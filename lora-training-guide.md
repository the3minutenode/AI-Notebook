## 1. Job & Model Configuration
This section defines the "who" and "where" of the training process.

* **Training Name:** A unique identifier for your project. This will typically be the folder name where your weights are saved.
* **GPU ID:** Specifies which graphics card to use. `GPU #0` is usually the primary card.
* **Trigger Word:** A specific keyword (e.g., "SKS style") that tells the model when to apply the learned LoRA during generation.
* **Model Architecture:** The base model type (e.g., Z-Image). LoRAs are architecture-specific; a LoRA trained for one model usually won't work on another.
* **Name or Path:** The local file path or Hugging Face repository name of the base model you are fine-tuning.
* **Low VRAM / Layer Offloading:** Performance toggles. **Low VRAM** reduces memory usage at the cost of speed. **Layer Offloading** moves parts of the model to your system RAM (CPU) to prevent crashes if your GPU is full.

## 2. Quantization & Target
Quantization is the process of compressing the model weights to save memory.

* **Transformer / Text Encoder Quantization:** These are set to `float8`. By reducing precision from 16-bit to 8-bit, you can fit much larger models onto consumer hardware with minimal loss in quality.
* **Target Type (LoRA):** Confirms you are training a Low-Rank Adaptation rather than a full fine-tune.
* **Linear Rank (32):** Also known as **dim**. It represents the complexity of the LoRA. Higher numbers (e.g., 64, 128) can capture more detail but result in larger file sizes and require more VRAM.

## 3. Save Settings
* **Data Type (BF16):** Bfloat16 is a high-performance format that maintains a wide range of values, preventing "gradient explosions" or NaN errors during training.
* **Save Every (250):** The frequency at which the trainer saves a checkpoint (e.g., every 250 steps). 
* **Max Step Saves to Keep (4):** To save disk space, the trainer will delete old checkpoints, keeping only the 4 most recent ones.

## 4. Training Hyperparameters
This is the "engine room" where you control how the model learns.

### Optimization
* **Batch Size (1):** How many images the model looks at before updating its internal "knowledge." Higher is more stable but requires significantly more VRAM.
* **Gradient Accumulation (1):** Simulates a larger batch size by running multiple steps before updating weights. 
* **Optimizer (AdamW8Bit):** The algorithm that calculates how to change weights. The **8-bit** version is highly efficient for consumer GPUs.
* **Learning Rate (0.0001):** The "speed" of learning. Too high and the model "breaks" (artifacts); too low and it learns nothing.

### Logic & Math
* **Steps (3000):** Total iterations. 3,000 is a standard starting point for medium-sized datasets.
* **Timestep Type/Bias:** Controls which parts of the "noise" the model focuses on (e.g., focusing on the early structural phase or the later detailing phase of image generation).
* **Loss Type (MSE):** Mean Squared Error. This is how the model measures its "mistakes" by comparing its output to your training data.

### Optimizations
* **EMA (Exponential Moving Average):** Creates a "smoothed" version of the model that is often more stable and less prone to noise.
* **Unload TE / Cache Text Embeddings:** Strategies to save VRAM by pre-calculating the text data and removing the Text Encoder from the GPU while the main model trains.

## 5. Datasets & Advanced
This defines what the model is actually looking at.

* **Num Repeats (1):** How many times the trainer sees each image in a single "epoch." 
* **Caption Dropout Rate (0.05):** A 5% chance the trainer ignores your captions. This forces the model to learn the *visuals* better, making it more flexible when you use different prompts later.
* **Cache Latents:** Pre-processes images into a compressed mathematical space before training starts. This significantly speeds up training.
* **Is Regularization:** If checked, these images are used as "anchors" to prevent the model from forgetting what a normal person/object looks like while learning your specific subject.
* **Resolutions:** These toggles tell the trainer which image sizes are allowed. Modern trainers use "Bucketing," allowing you to use a mix of portrait, landscape, and square images.

### Pro-Tip for setup:
If you have **Low VRAM** enabled, you're likely on a card with 8GB–12GB. If you experience crashes, try increasing **Gradient Accumulation** to 2 or 4 and reducing **Batch Size** if it was higher than 1.
