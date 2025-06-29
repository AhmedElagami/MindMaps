# 🧠 Supervised Learning & Neural Networks
## 🧹 Data Preprocessing

### ~"N → Narrow range, S → Stats adjust!"
### 🔄 Normalization
- Scales data to a fixed range, typically **[0, 1]** - **RANGE BASED**
- Use when data range is arbitrary and **does not follow a normal (Gaussian) distribution**
- Good for algorithms that rely on **bounded input** (e.g., KNN, Neural Nets with sigmoid/tanh)

### ⚖️ Standardization
- Transforms data to have **mean = 0** and **standard deviation = 1** - **DISTRIBUTION BASED**
- Use when data **follows a normal distribution**
- Helps when comparing features with **different units/scales**

## 📊 Regression
### Simple & Multiple Regression
- Simple: one independent feature $x$, Multiple: multiple independent features $x_i$
- Goal: predict the dependent target $y$ from $x_i$
### Loss Functions
- Measures how well a model's predictions match the actual values
- MSE (Mean Squared Error): $$\text{MSE} = \frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2$$

- MAE (Mean Absolute Error): $$\text{MAE} = \frac{1}{n} \sum_{i=1}^{n} |y_i - \hat{y}_i|$$

- RMSE (Root Mean Squared Error):	$$\text{RMSE} = \sqrt{ \frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2 }$$

## 😁 Neural Networks
### Neuron & Layers
- Neuron: 
  $$y = f\left(\sum_{i=1}^{m} w_i x_i + b\right)$$
- Components: Inputs, Weights, Bias, Activation Function, Output
- Organized as layers: Input → Hidden(s) → Output
- Each layer extracts and combines features at a higher level of abstraction.
- ![](attachments/Pasted%20image%2020250629171631.png)
- ![](attachments/Pasted%20image%2020250629171637.png)
### Deep Neural Networks
* **Loss function**: $L(h(x), y)$
  * Measures the error between the model’s prediction $h(x)$ and the true label $y$
  * Applied **per sample**
* **Cost function**: $C(\theta)$ or $J(W)$
  * Aggregates the loss across the **entire training dataset**
  * $\theta$ or $W$ represent the model’s learnable parameters (e.g., weights)
### ⚙️ Hyperparameters vs Parameters
- **Hyperparameters**:
    - Set **before training**
    - Control the learning process and model architecture
    - Examples:
        - Number of layers
        - Number of neurons per layer
        - Learning rate
        - Batch size
        - Activation functions
- **Parameters**:
    - **Learned during training** by minimizing the loss function
    - Include:
        - **Weights** $W$
        - **Biases** $b$
    - Initially set (often **randomly**) and then updated via **gradient descent**
### 💪 Activation Functions
#### Purpose
- Introduce **non-linearity** into the network, enabling it to learn **complex, non-linear patterns & relationships** between inputs and outputs
- Without activation functions, an ANN would act like a **linear model**, regardless of how many layers it has

#### 🔢 Common Activation Functions and Ranges
- **Sigmoid**: $(0, 1)$ — good for binary classification
- **Tanh**: $(-1, 1)$ — zero-centered; stronger gradients
- **ReLU**: $[0, \infty)$ — efficient and widely used in hidden layers
- ![](attachments/Pasted%20image%2020250629172934.png)

#### 🧠 Example Use Case
- An ANN can take features like **height** and **weight** as input
- It uses **activation functions** in hidden layers to extract patterns
- The final output neuron (e.g., with Sigmoid) predicts a value:
    - Close to **0** → Human
    - Close to **1** → Animal
 
### 🛠 Optimization & Optimizers (Finding Optimal Parameters)
#### Analytical Solution to find optimal parameters
- Solves the equation directly using matrix algebra; **Exact Solution**.
- Normal Equation: 
  $$\theta = (X^T X)^{-1} X^T y$$

#### ⚙️ Gradient Descent (GD)
* Iteratively updates parameters to minimize the loss
* **Approximate solution** that scales better with large data
* **Core Steps**:
  * Initialize weights randomly $\theta$
  * → Predict $\hat{y}$
  * → Compute loss $L(\hat{y}, y)$
  * → Compute gradients $\nabla L$
  * → Update $\theta = \theta - \eta \nabla L$
  * → Repeat until convergence
* **Update rule**:
  $\theta = \theta - \eta \nabla L$
* **Learning rate** $\eta$: step size for updating weights
#### 🔁 GD Variants
* **Momentum**:
  * Adds velocity from previous updates
    $v_t = \gamma v_{t-1} + \eta \nabla L$
    $\theta = \theta - v_t$
  * Reduces oscillations and speeds up convergence
* **Adam** (Adaptive Moment Estimation):
  * Combines **momentum** and **adaptive learning rates**
  * Maintains exponentially decaying averages of gradients (1st moment) and squared gradients (2nd moment)
  * **Fast convergence**, good for **sparse gradients** or **non-stationary objectives**
#### 🧱 Batch Size & GD Types
##### **Batch Size**
###### Is the number of samples processed before each weight update
###### Affects **speed, stability, and memory usage**
######
![](attachments/Pasted%20image%2020250629171810.png)
##### 📊 Batch Gradient Descent
* Uses the **entire dataset** to compute gradients
* **Batch size = total number of training samples**
* Stable updates but **slow for large datasets**
##### ⚖️ Mini-Batch Gradient Descent
* Uses **small random subsets** (e.g., 16, 32, 64 samples)
* Combines benefits of both batch and stochastic methods
* Most commonly used in practice
* Mini-batches are **reshuffled** every epoch
##### 🎲 Stochastic Gradient Descent (SGD)
* Updates weights using **a single training example**
* **Batch size = 1**
* **Fast and memory-efficient**, but updates are noisy - Introduces randomness → helps generalization
* Good for escaping local minima
* One **epoch** = one full pass through the dataset

## 🔁 Backpropagation
### Overview
- Core algorithm for training neural networks using **gradient-based optimization**
- Involves two main passes:
    - ➕ Forward Pass
	    - Computes the output $\hat{y}$ and loss $L(\hat{y}, y)$
	    - Purpose: evaluate how well the model performs
    - ➖ Backward Pass
	    - Applies the **Chain Rule of Calculus** to compute gradients $\nabla L$ w.r.t. each weight
	    - Purpose: determine how to adjust weights to minimize loss
	    - Updates weights using an optimizer (e.g., GD, SGD, Adam)
- **Epoch**: one full pass through the training dataset (forward + backward)
- Optimization is typically performed using **GD variants** like SGD, Momentum, or Adam
    

### 🔧 PyTorch Autograd

- PyTorch uses a **Dynamic Computational Graph (DAG)** to track operations during the forward pass
- **Nodes in the graph**:
    - **Leaf nodes**: Input tensors (with `requires_grad=True`)
    - **Intermediate nodes**: Functions and operations applied to tensors
    - **Root node**: Final scalar output (typically the **loss**)
- The **backward pass** is triggered by:
    - Calling `.backward()` on the **loss tensor**
    - Autograd then computes gradients by traversing the DAG in reverse using the **Chain Rule**
        
## 🔢 Classification
### Predicts a **category or class label**, unlike regression which outputs a **continuous value**
### Example tasks:
  * **Binary Classification**: Human vs Animal
  * **Multi-Class Classification**: Dog, Cat, Horse, etc.
  * **Image Classification**: Assigns an image to a predefined category
  
### ✅ Binary Classification
#### 🎯 Targets and Goal
* **Target variable**: $y \in {0, 1}$
* **Goal**: Learn a function $\hat{y} = f(x)$ that predicts class membership
* **Output formula**:$$\hat{y} = \sigma(w \cdot x + b)$$
#### ⚙️ Activation & Output
* Uses **Sigmoid activation**:
  $\sigma(z) = \frac{1}{1 + e^{-z}}$
- ![](attachments/Pasted%20image%2020250629171511.png)
* Produces **probabilities** in range $(0, 1)$
#### 📉 Loss Function
* **Binary Cross-Entropy**:$$L = -[y \log(\hat{y}) + (1 - y) \log(1 - \hat{y})]$$
- ![](attachments/Pasted%20image%2020250629170541.png)
* **Clipping** may be applied to avoid $\log(0)$ numerical issues
#### 🧠 Network Structure
* **Simple**: Single neuron (logistic regression)
* **Deep**: Multi-layer neural networks capture more complex patterns
### 🟨 Multi-Class Classification
#### 🎯 Outputs and Activation
* **$K$ output neurons**: One per class
* **Activation function**: **Softmax**, which outputs class probabilities:
  $$
  \hat{y}_i = \frac{e^{z_i}}{\sum_{j=1}^K e^{z_j}}
  $$
* Output vector:
  * Each $\hat{y}\_i \in (0, 1)$
  * $\sum \hat{y}\_i = 1$
  * Represents a **probability distribution**
- ![](attachments/Pasted%20image%2020250629171950.png)
#### 🏷️ Ground Truth Encoding
* Labels use **one-hot encoding**
* Example ($K=3$ classes):
  * Cat: $[1, 0, 0]$
  * Dog: $[0, 1, 0]$
  * Horse: $[0, 0, 1]$
- ![](attachments/Pasted%20image%2020250629171902.png)
#### 📉 Loss Function
* **Categorical Cross-Entropy**:
  $$
  L = -\sum_{i=1}^{K} y_i \log(\hat{y}_i)
  $$
## 📸 CNNs (Convolutional Neural Networks)
### 🧠 Purpose & Advantages
- CNNs automatically extract **local spatial features** from images using convolutional filters
- Efficient due to **shared weights** and **sparse connectivity**
- Ideal for images because of:
    - **Translation Equivariance**: shifted input → shifted output
    - **Local feature extraction**: detects patterns like edges, textures
### 🧱 Core Components
- **Convolutional Layers**: apply filters (kernels) to detect features
	- ![](attachments/Pasted%20image%2020250629225752.png)
- **Pooling Layers**: reduce spatial size, retain most important features (e.g., Max Pooling)
- **Fully Connected (FC) Layers**: final decision layers
- ![](attachments/Pasted%20image%2020250629172108.png)
### ⚙️ Convolution Operation
- A filter slides over the input and computes a **weighted sum**
- **Output size formula**:$$\text{Output size} = \left( \frac{n + 2p - f}{s} + 1 \right)$$
	- $n$: input size
    - $f$: filter size
    - $p$: padding
    - $s$: stride
    - $c_o$: number of output channels
- **Example**:  
    Input: $224 \times 224 \times 3$  
    Filter: $3 \times 3 \times 3$, $p = 0$, $s = 1$, $c_o = 32$  
    → Output: $222 \times 222 \times 32$
- **Weights in filters** are learnable and function like those in FC layers but are spatially arranged
- After convolution, a **bias term** is added before applying activation
### 🧰 Padding & Stride
- **Padding**:
    - _Valid_: $p = 0$ (no padding) → smaller output
    - _Same_: $p = \lfloor f/2 \rfloor$ → preserves input size
    - Prevents loss of edge information
- **Stride**:
    - $s=1$: filters move by 1 pixel → detailed output
    - $s=2$: filters skip pixels → coarser, faster output
### 📏 Output Depth
- **More filters = deeper output**
- Each filter detects a different feature
- Output volume: $H_{\text{out}} \times W_{\text{out}} \times C_{\text{out}}$, where $C_{\text{out}}$ = number of filter
### 🔄 Pooling Layers
- **Max Pooling**: selects the max value from a region (e.g., $2 \times 2$)
- Example:      Input: $3 \times 3$, Filter: $2 \times 2$, Stride: 2      → Output: $1 \times 1$
- Helps achieve **Translation Invariance**: small shifts in input ≈ stable output
### 🔌 Activation Functions
- Common in CNNs:
    - **ReLU**: default for hidden layers - fast, avoids vanishing gradient
		- ![](attachments/Pasted%20image%2020250629172222.png)
    - **Sigmoid**: rarely used in conv layers, more common at output for binary tasks

### 🏗 Architectures
#### ⏳ Historical Milestones
* **Neocognitron** (Fukushima, 1980): early visual pattern model
* **LeNet-5** (1998): digits
* **AlexNet** (2012): ImageNet breakthrough
* **VGG16** (2014): stacked $3 \times 3$ filters
* **Inception / GoogLeNet**: parallel filters (1×1, 3×3, 5×5)
#### 📌 VGG16 Highlights
* Stacked small filters → deeper, simpler design
#### 🧬 Inception Highlights
* Parallel filters at each layer
* 1x1 conv for dimensionality reduction
* Auxiliary classifiers during training
#### 🧠 Feature Learning with Convolutions
* Increasing **depth with small filters** (e.g., stacked $3\times3$) builds up **non-linear decision boundaries**
* Hierarchy of learned features:
  * **Low layers**: edges, corners
  * **Mid layers**: textures, object parts
  * **High layers**: full objects (e.g., faces)
#### 📌 VGG16: Why 3×3 Matters
* Replaces larger $5\times5$ or $7\times7$ filters with **stacked $3\times3$**
* Adds **non-linearity** between layers
* Increases **capacity** while reducing parameters
#### 🧠 ResNet (2015, He et al.)
* Introduces **residual learning**:
  $\text{Output} = F(x) + x$
* Focuses on learning changes (residuals) rather than full transformations
* **Identity mappings**: allow deeper blocks without degrading performance
* Enables very deep networks to train effectively (e.g., ResNet-50, ResNet-101)
#### 🔧 Other Modern Architectures
* **DenseNet (2016)**: layer-wise connections for feature reuse
* **MobileNet (2017)**: lightweight, uses **depthwise separable convolutions**
* **MobileNet v2 (2018)**: adds inverted residuals
* **EfficientNet**: scales width/depth/resolution using a compound coefficient
* **ConvNeXt**: CNN-based transformer alternative
* **ViT**: Vision Transformers — transformer applied to image patches
#### 🔍 MobileNet Deep Dive
* Combines:
  * **Depthwise convolution**: per-channel spatial filtering
  * **Pointwise (1x1) convolution**: recombines features
* Result: drastic reduction in computation with competitive performance
### ⚠️ Deep CNN Challenges
#### 🔺 Overfitting vs Underfitting
* **Underfitting**: use deeper nets or more data
* **Overfitting**: apply regularization, dropout, or augmentation
* Smaller models → less prone to overfitting
#### 🌀 Gradient Issues
* **Vanishing Gradient**:
  * Gradients become **very small** as they are backpropagated.
  * Earlier layers (closer to input) **learn very slowly** or **not at all**.
  * Common with **sigmoid** or **tanh** activations.
* **Exploding Gradient**:
  * Gradients become **very large**, causing **unstable updates**.
  * Leads to **numerical overflow** or **NaN loss**.
  * Can cause weights to oscillate or diverge
- ![](attachments/Pasted%20image%2020250629172725.png)
* **Solutions**:
  * Proper weight initialization
  * **Batch normalization**
  * **ReLU** activations
  * **Gradient clipping**
### ⚙️ Weight Initialization
* **Purpose**: set starting values for weights
* **Distributions**: normal, uniform
* 🔧 Strategies
  * **Conv layers**: He (Kaiming) initialization
  * **Linear layers**: Xavier (Glorot) initialization
  * **Normal init**: $\mu = 0$, small $\sigma$ (e.g., 0.01)
### 🧹 Regularization Techniques
#### 📦 Types
* **L1 (LASSO)**: $\lambda \sum |w\_i|$ → sparse weights
* **L2 (Ridge)**: 
$\lambda \sum w\_i^2$ → penalizes large weights
* **Dropout**: disables random neurons during training
* **Weight decay**, **Early stopping**
#### 🔥 Dropout
* Prevents co-adaptation
* Acts like training many subnetworks
* Encourages generalization and redundancy
### 🔄 Batch Normalization
* Normalizes activations per mini-batch
* Tracks running mean & variance
* Applies learnable scale & shift: $\gamma$, $\beta$
* Helps with stability and convergence speed
### 📚 ReLU vs Sigmoid/Tanh
* **ReLU**: faster training, avoids vanishing gradients
* No need for input normalization
* Only active for positive inputs
### 📊 ImageNet & Evaluation
* **ImageNet Challenge**: benchmark for image classification
* Dataset: **1.2M images**, **1000 classes**
### 🧠 FC vs CNN
- **Fully Connected (FC) Layers**:
    - Every **input neuron is connected to every output neuron** using **learned weights and biases**
    - Captures global patterns but is **parameter-heavy** and **not spatially aware**
- **CNN Layers**:
    - Use **local filters** to connect only nearby neurons
    - More efficient and preserves spatial structure
    - Suitable for image and spatial data


## 📈 Transfer Learning
### 🔹 Basics & Techniques
- **Two main parts of the model**:
    - **Feature extractor** (also called **Backbone**): convolutional base
    - **Classifier** (also called **Head**): fully connected head or custom layers
- **When to use Transfer Learning**:
    - Target dataset is **small**
    - **Training from scratch** is not feasible
    - **Pre-trained model** exists on a **similar domain**
- **Effect on training speed**:
    - Using pre-trained weights **speeds up training significantly**
### 🔍 Backbone vs Head

####

| Component    | Role                                        | Typical Training Strategy        | Examples                         |
| ------------ | ------------------------------------------- | -------------------------------- | -------------------------------- |
| **Backbone** | Extracts general features (edges, textures) | Often **frozen** (not trained)   | ResNet, VGG, EfficientNet        |
| **Head**     | Performs task-specific prediction           | Usually **trained from scratch** | Dense layers, custom classifiers |
#### ![](attachments/Pasted%20image%2020250629173355.png)
### 🔹 Modes of Transfer Learning
#### 🧲 Feature Extraction
* **Initialization**: use **pretrained weights** (e.g., ImageNet)
* **What is frozen**: convolutional base
* **What is trained**: new classifier head
* **Use case**: small datasets, fast training
- ![](attachments/Pasted%20image%2020250629173406.png)
#### 🔧 Fine-Tuning
* **Initialization**: use full **pretrained model**
* **What is trained**: the **entire network** (base + classifier)
* **Pretraining dataset**: usually **ImageNet**
* **Use case**: large datasets, more compute, better results
- ![](attachments/Pasted%20image%2020250629173442.png)
### 🔍 Performance Comparison Summary

| Model | Method         | Accuracy | Speed  | Notes                     |
| ----- | -------------- | -------- | ------ | ------------------------- |
| **B** | Feature on B   | Moderate | Fast   | Few layers trained        |
| **C** | Fine-tune on B | Higher   | Slower | Full model retrained      |
| **D** | Feature on C   | Lower    | Fast   | Good for small datasets   |
| **E** | Fine-tune on C | Best     | Slow   | Best if C is large enough |
## 🔄 Autoencoders
### 🔹 Fundamentals
* An **Autoencoder** is a **neural network** that learns to compress and reconstruct its own input
* Used in **unsupervised/self-supervised learning** → no labels required (input = target)
* Commonly trained using **backpropagation** to minimize **reconstruction loss** (e.g., MSE)
### 🔹 Architecture
* Consists of **three parts**:
* **Encoder**: $f(x)$ → compresses input into latent representation
* **Latent Space**: compressed intermediate form ($z$)
* **Decoder**: $g(z)$ → reconstructs input: $\hat{x} = g(f(x))$
* The **latent space** is:
	* Lower-dimensional than input
	* Encodes **key features** of the data
	* Forms a **low-dimensional manifold**
- ![](attachments/Pasted%20image%2020250629173546.png)
### 🔹 Training & Limitations
* **Loss Function**: $$\text{MSE} = \frac{1}{n} \sum_{i=1}^{n} (x_i - \hat{x}_i)^2$$

* Can **overfit** and fail to generalize on real-world data
* Not ideal for **true compression** tasks compared to engineered algorithms
### 🧹 Types of Autoencoders

#### 🔸 Convolutional Autoencoders
* Tailored for **image data**
* Encoder: **Conv + Pooling**
- Decoder: **Transposed Conv + Upsampling**
* Good at capturing **spatial patterns**
* Typical drawback: **blurry reconstructions**
- ![](attachments/Pasted%20image%2020250629173651.png)
#### 🔸 Denoising Autoencoders
* Trained to remove **noise** from corrupted input
* Input = noisy version, Output = clean image
* Encourages learning **robust, noise-invariant** features
* Noise types: **Gaussian**, **salt-and-pepper**, **random masking**
* ![](attachments/Pasted%20image%2020250629173701.png)
#### 🔸 Sparse Autoencoders
* Enforces **sparsity** in the hidden layer
* Penalizes non-zero activations (e.g., KL divergence)
* Helps prevent **memorization** and overfitting
* Promotes learning of **distinct, useful features**
* ![](attachments/Pasted%20image%2020250629173721.png)
#### 🔸 Variational Autoencoders (VAEs)
* **Probabilistic & generative** autoencoders
* Learn **mean** and **variance** for each latent dimension
* Enable **sampling** from latent space to generate new data
* Used in **generative modeling**, **anomaly detection**, and **latent interpolation**
* ![](attachments/Pasted%20image%2020250629173738.png)
### 🧪 Applications
* **Dimensionality reduction** (unsupervised alternative to PCA)
* **Feature extraction** (feed latent vector into another model)
* **Anomaly detection** (high reconstruction error = anomaly)
* **Image denoising**
* **Generative modeling** (especially with VAEs)

## 🔪 Object Detection
### 🔍 What is Object Detection?
- To identify **what** objects are present in an image and **where** they are located by drawing bounding boxes and classifying each one.
#### 🆚 Classification vs Detection
- **Classification** assigns a single label to the entire image.
- **Detection** identifies and locates multiple objects with bounding boxes and class labels.
#### 📦 Required Outputs per Object
- **Label**
- **Bounding Box**: $(x, y, w, h)$
- **Confidence score**
#### 📐 Bounding Box Format
- $x$, $y$: top-left corner position
- $w$: width
- $h$: height
#### 🖼️ Image Resolution Needs
- High resolution preserves fine details for small or overlapping objects.
### 🧭 Types of Detection Tasks
#### 🎯 Classification + Localization
- Classifies a single object and predicts its bounding box.
#### 🧃 Naïve Approach: Sliding Window
- Scans the image patch-by-patch to detect objects.
### 🏗️ Object Detection Architectures

#### 🧱 Two Main Categories
- **Two-Stage Detectors**: R-CNN, Fast R-CNN, Faster R-CNN
- **One-Stage Detectors**: SSD, YOLO
#### 🧩 Two-Stage Detectors
##### 📦 R-CNN
- Uses **Selective Search** for region proposals.
- Applies CNN separately to each region.
- Adds **bounding box regression**.
- ![](attachments/Pasted%20image%2020250629174140.png)
##### ⚡ Fast R-CNN
- CNN is applied **once** to the whole image.
- **ROI Pooling** extracts features.
- Much faster than R-CNN.
- ![](attachments/Pasted%20image%2020250629174235.png)
##### 🚀 Faster R-CNN
- Replaces Selective Search with a **Region Proposal Network (RPN)**.
- Learns to propose regions using shared features.
- More efficient but **not real-time**.
- ![](attachments/Pasted%20image%2020250629174301.png)
#### ⚡ One-Stage Detectors
##### 🟨 YOLO (You Only Look Once)
- A single CNN directly predicts bounding boxes and classes.
- ![](attachments/Pasted%20image%2020250629174437.png)
###### YOLO V2
- Batch normalization
- K-means anchor boxes
- Pretrained classifier
###### YOLO V3
- Multi-scale predictions
- Residual connections
- Improved classification
##### 🔳 SSD (Single Shot Detector)
- Predicts classes and boxes from multiple feature maps.
- Uses **default anchor boxes** with various aspect ratios.
- ![](attachments/Pasted%20image%2020250629174403.png)
##### 📐 YOLO vs SSD (Anchor Boxes)
- **YOLO**: Learns anchors via k-means.
- **SSD**: Uses predefined boxes.
### 📏 Post-Processing Techniques
#### 🔲 IoU (Intersection over Union)
- IoU = Area of Overlap / Area of Union
- ![](attachments/Pasted%20image%2020250629174614.png)
#### 🚫 Non-Maximum Suppression (NMS)
- Keeps the **highest confidence** box.
- Suppresses overlapping boxes.
- ![](attachments/Pasted%20image%2020250629174651.png)
#### ❌ Problems with NMS
- Suppresses valid overlapping objects.
- Harsh thresholds hurt detection accuracy.
#### ✅ Alternatives to NMS
- **Soft-NMS**: Decays confidence instead of removing.
- **Learned NMS**: Learns suppression behavior.
- **Dynamic IoU Thresholds**: Adjusts per class/context.
### 📊 Evaluation Metrics
#### ✅ Primary Metric: mAP (Mean Average Precision)
- Area under precision-recall curve
- Averaged over all classes
#### 📏 AP Variants
- **AP50**: IoU threshold = 0.5
- **AP75**: IoU threshold = 0.75
#### 🔍 AP by Object Size
- **APS**: Small
- **APM**: Medium
- **APL**: Large
### 🧮 Model Complexity
- **Params (M)**: Total model parameters
- **GFLOPs**: Giga Floating Point Ops (inference cost)
## 🤹 Image Segmentation
### 🔍 Types of Image Segmentation
#### Semantic Segmentation
* Assigns a class label to **each pixel**.
* Does **not differentiate** between object instances.
- ![](attachments/Pasted%20image%2020250629174915.png)
#### Instance Segmentation
* Segments **each object instance** separately.
* Differentiates overlapping objects of the same class.
* ![](attachments/Pasted%20image%2020250629174923.png)
#### Panoptic Segmentation
* Combines **semantic + instance segmentation**.
* Every pixel is assigned:
* A **semantic label** (e.g., sky, road).
* An **instance ID** (e.g., car 1, car 2).
* ![](attachments/Pasted%20image%2020250629174857.png)
### 🏗️ Architectures
#### 🔧 Encoder-Decoder (e.g., Convolutional Autoencoders)
* Encoder: conv + pooling
* Decoder: upsampling (unpooling, transposed conv)
* Strength: Captures local patterns
* Weakness: Poor detail recovery
#### 🔄 Fully Convolutional Networks (FCN)
* End-to-end architecture (no dense layers)
* Uses transposed convolutions for upsampling
* Skip connections improve detail
* Output often coarse
#### 🔬 U-Net
* Symmetric architecture: contracting + expanding path
* Skip connections at each level
* Works well on biomedical images
* Many variants: Residual U-Net, Attention U-Net
#### 🚀 DeepLab (v1-v4)
* **Dilated convolutions** to capture context
* **ASPP** for multi-scale context
* **CRFs** for boundary refinement
#### 🧠 Mask R-CNN (Instance Segmentation)
* Faster R-CNN + mask prediction head
* Outputs binary mask for each detected object
* Uses **RoIAlign** for precise feature extraction
- ![](attachments/Pasted%20image%2020250629175008.png)
#### 🪄 Segment Anything Models (SAM, SAM2)
##### 

| Model | Input   | Output                  |
| ----- | ------- | ----------------------- |
| SAM   | Prompts | Binary masks            |
| SAM2  | Prompts | Masks + semantic labels |
##### SAM: Prompt-based segmentation (click, box, mask)
##### SAM2: Adds **semantic understanding** to masks

### 🎯 Applications
#### Image
* Medical imaging (e.g., tumor detection)
* Autonomous driving
* Satellite analysis
* Industrial inspection
* Image editing
#### Video
* Real-time object tracking & segmentation
* Used in surveillance, robotics, AR
### 🧪 Core Tasks
#### Classification
* One label per image
#### Reconstruction
* Encoder → Latent Space → Decoder
* Applications: denoising, inpainting
* Limitation: Focuses on pixel appearance, not meaning
### 🔄 Upsampling Techniques
#### Unpooling
* Uses pooling indices (non-learnable)
* Simple but limited
#### Transposed Convolution
* Learnable; may cause checkerboard artifacts
* Formula:$$O = (I - 1) \cdot S - 2P + K$$

#### Interpolation
* Bilinear, nearest-neighbor (non-learnable)
### 📉 Loss Functions
#### Cross-Entropy Loss
* $CE = -\log(p\_{true})$
* Biased toward frequent classes
#### Dice Loss
* $$\text{Dice} = \frac{2TP}{2TP + FP + FN}, \quad \text{Loss} = 1 - \text{Dice}$$
* Handles class imbalance better
### 🧪 Training & Evaluation
1. Data collection
2. Preprocessing
3. Model design
4. Loss selection
5. Training
6. Validation and tuning

### 📋 Summary & Comparisons
| Feature         | FCN                    | U-Net                   | Mask R-CNN               | SAM / SAM2                       |
| --------------- | ---------------------- | ----------------------- | ------------------------ | -------------------------------- |
| Architecture    | Encoder-decoder        | Symmetric with skips    | Faster R-CNN + mask head | Prompt-based segmentation        |
| Granularity     | Semantic               | Semantic                | Instance                 | Binary (SAM) / Semantic (SAM2)   |
| Label Output    | Class per pixel        | Class per pixel         | Class + instance masks   | No (SAM) / Yes (SAM2)            |
| Upsampling      | Transposed convolution | Transposed + skip       | RoI-aligned FCN          | None (pre-computed embeddings)   |
| Main Limitation | Coarse maps            | Moderate generalization | Complexity, speed        | No class labels (SAM), black-box |

## 🌐 Sequential Data & RNNs
### 🔗 Local Relationships
* **Nearby elements** in sequences (like words or time steps) tend to be more **strongly related**
* Models like RNNs and CNNs use **local connectivity** to efficiently learn from these short-range dependencies
* **Local relationships** allow for faster and more interpretable computations
* **Distant elements** have **weaker dependencies**, making long-range context harder to model
* Deep or attention-based models (like LSTM, Transformer) are better suited for capturing **long-range patterns**
* More complex structures can reduce **transparency** and increase **training time**
### 📌 What is Sequential Data?
* **Sequential data** is data where the **order of elements matters**.
* Examples:
	* Natural language (e.g., grammar)
	* Time series (e.g., stock prices)
	* Audio signals (e.g., waveform)
	* DNA sequences (e.g., nucleotide order)
	- Video frames
	- Biological sequences (e.g., proteins)
	- Sensor readings (time-series from IoT, etc.)
* Nearby elements = strong dependency
* Distant elements = weak dependency, harder to model
* Deep models (LSTM, Transformer) help capture long-term context
* Simple models = faster but less expressive
- 🔸 Properties
	* **Order matters**
	* **Context-dependent**: surrounding elements affect interpretation
	* **Variable-length**
	* **Temporal dependencies** Earlier inputs influence later ones
	* **Local vs. long-range relationships**:
### 🧠 Model Requirements for Sequential Data
* Have **memory** of past inputs
* Understand **context**
* Handle **variable-length sequences**
* Be **position-sensitive**	
- Flexibility with input-output lengths (e.g., One-to-Many)

### 🎯 Tasks on Sequential Data
#### 🔹 Named Entity Recognition (NER)
* **Input**: `["Barack", "Obama", "was", "born", "in", "Hawaii"]`
* **Output**: `["PER", "PER", "O", "O", "O", "LOC"]`
#### 🔹 Task Categories in RNNs
| Type           | Example              |
| -------------- | -------------------- |
| One-to-One     | Image classification |
| One-to-Many    | Image captioning     |
| Many-to-One    | Sentiment analysis   |
| Many-to-Many ✔ | NER                  |
| Many-to-Many ✖ | Machine translation  |
#### 🔹 Common NLU Tasks
- Sentiment analysis
- Named Entity Recognition (NER)
- Question answering
- Machine translation
- Text classification
- Coreference resolution

### 🔠 Word Representations
#### 🔸 One-Hot Encoding
* Long sparse vectors (e.g., `[0, 1, 0]`)
* Limitations:
  * No semantic meaning
  * High dimensionality
#### 🔸 Word Embeddings
* Dense, low-dimensional vectors (e.g., 300D)
* Capture semantic similarity
* Examples: **Word2Vec (CBOW, Skip-gram)**, **GloVe**, which are learned by predicting context words or co-occurrence patterns.
* Visualized using **t-SNE** (2D/3D plots) that preserve local structure and reveals clustering of similar words
* Beyond text: used in **images (CNN features)**, **audio (spectrograms)**, **graphs (GNNs)**, **recommender systems**
### 🤖 Processing Sequential Data with Neural Networks
#### 🔸 Why Standard NN Fails
* Fixed-size input/output
* No memory
* Cannot model time dependencies
### 🔁 Recurrent Neural Networks (RNNs)
* Maintains hidden state
* Step-by-step processing
* Local connectivity helps with nearby elements
- ![](attachments/Pasted%20image%2020250629175453.png)
- 🔹 RNN Formulas
	* Hidden state:  $$  h_t = \tanh(W_{hh}h_{t-1} + W_{xh}x_t + b_h) $$
	- ![](attachments/Pasted%20image%2020250629175306.png)
	* Output:  $$  y_t = W_{hy}h_t + b_y $$
- 🔹 Training (BPTT)
	- Forward pass
	- Backward pass (unroll through time)
	- Gradient descent
- ⚠️ Problems with Vanilla RNNs
	* **Vanishing gradients**: can't learn long-term dependencies
	* **Exploding gradients**: instability during training
	* ![](attachments/Pasted%20image%2020250629175445.png)
### 🧬 Long Short-Term Memory (LSTM - Solution to Vanishing Gradient)
#### **Components**

| Component    | Function                |
| ------------ | ----------------------- |
| Cell state   | Long-term memory        |
| Hidden state | Output per step         |
| Input gate   | Allow new input         |
| Forget gate  | Discard irrelevant info |
| Output gate  | Control what to output  |
####
![](attachments/Pasted%20image%2020250629175628.png)
#### Adds **memory cell**, **input**, **forget**, and **output gates**
#### Uses **sigmoid and tanh** activations
####
![](attachments/Pasted%20image%2020250629175536.png)
### 🧮 Gated Recurrent Unit (GRU)
#### Simplified LSTM:
  * No cell state
  * Two gates: **update** and **reset**

####
![](attachments/Pasted%20image%2020250629175714.png)
#### LTSM vs GRU

| Feature        | LSTM   | GRU    |
| -------------- | ------ | ------ |
| Gates          | 3      | 2      |
| Cell State     | Yes    | No     |
| Simplicity     | Less   | More   |
| Training Speed | Slower | Faster |

### 🔄 Encoder-Decoder (Seq2Seq)
#### Used in **machine translation**, etc.
#### Architecture
* **Encoder** encodes sequence to vector `z`
* **Decoder** generates output using `z`
- ![](attachments/Pasted%20image%2020250629175822.png)
- ![](attachments/Pasted%20image%2020250629175829.png)

####
| Role    | Description                      |
| ------- | -------------------------------- |
| Encoder | Maps input sequence to vector    |
| Decoder | Uses vector to generate output   |
| z       | Compressed latent representation |
#### ⚠️ Limitations
* Fixed-size `z` loses information
* Poor for long sequences
* **Context loss** → degraded translation

### 🎯 Attention Mechanism
####
* Overcomes bottleneck in seq2seq
* Allows **decoder** to focus on **relevant encoder states**
* Replaces dependence on single `z` vector
#### 🔸 Mechanism
1. Compute attention scores:   $$   e_{t,t'} = \text{score}(s_t, h_{t'})   $$
2. Apply softmax:   $$   \alpha_{t,t'} = \frac{\exp(e_{t,t'})}{\sum_k \exp(e_{t,k})}   $$
3. Context vector:  Weighted sum of encoder outputs
4. ![](attachments/Pasted%20image%2020250629175947.png)
#### 🔸 Benefits
* Preserves context
* Enables handling of long sequences
* Dynamic focus per decoder step
### 🔺 Transformers (bonus context)
#### Definition
* No recurrence → uses **self-attention**
* GPT: **unidirectional**
* BERT: **bidirectional**
* Supports large context windows, fast parallel training
#### 📐 Positional Encoding in Transformers
- Transformers are non-recurrent and need positional signals
- Positional Encoding adds sine/cosine-based patterns to token embeddings:  $$PE_{(pos, 2i)} = \sin(pos / 10000^{2i/d})\\PE_{(pos, 2i+1)} = \cos(pos / 10000^{2i/d})$$
- Makes each position unique while keeping embedding dimensions compatible
#### 🔹 Self-Attention Computation
- For each token embedding $x$:
  - Query: $Q = W_Q x$
  - Key: $K = W_K x$
  - Value: $V = W_V x$
- Output is computed as:  $$  \text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right) V  $$
- Captures relationships between all tokens at once
#### 🤖 GPT (Generative Pre-trained Transformer)
- **Unidirectional** (left-to-right) language model
- Uses **masked self-attention** to prevent peeking at future tokens
- Training:
  1. **Pre-training**: Predict next word
  2. **Fine-tuning**: Adapt to tasks like QA, summarization

#### 🧠 BERT (Bidirectional Encoder Representations from Transformers)
- **Bidirectional** self-attention: looks both left and right
- Pretrained with:
  - **Masked Language Modeling (MLM)**
  - **Next Sentence Prediction (NSP)**
- Special token `[CLS]` used for classification
#### 📊 Evaluation: BLEU Score (for Seq2Seq Tasks)

##### 🔹 What is BLEU?
- Bilingual Evaluation Understudy
- Measures quality of machine translation

##### 🔹 Calculation
- ![](attachments/Pasted%20image%2020250629180301.png)
##### 🔹 Key Features
- Precision over **n-grams**
- Penalizes overly short outputs (brevity penalty)
- **Clipped precision** avoids rewarding repeated phrases

##### 🔹 N-gram Example - Sentence: *"The cat sat on the mat"*
| N-gram | Examples |
|--------|----------|
| 1-gram | The, cat, sat, on, the, mat |
| 2-gram | The cat, cat sat, ..., the mat |
| 3-gram | The cat sat, cat sat on, ..., on the mat |


## 🔮 Vision Transformers & DETR
### ✂️ DEtection TRansformer (DETR)
#### 🔹 Introduced
- **Year**: 2020
- **Full name**: DEtection TRansformer
#### 🔹 Architecture
- **CNN backbone** (e.g., ResNet) for feature extraction
- **Transformer** for processing image features
- **Feedforward network (FFN)** to predict bounding boxes and class labels
#### 🔹 Detection Pipeline
- **No region proposals**, **no anchors**, **no NMS**
- Uses **bipartite matching** (Hungarian algorithm) to match predictions with ground-truth
- Enables **end-to-end object detection** using transformers
#### 🔹 Extensions
- Can be extended for **Instance Segmentation** by adding a **mask head**
#### 🔹 Summary Characteristics
- Transformer-based object detection
- No anchors or NMS
- Simplified pipeline with **global reasoning**
### 🧠 Vision Transformers (ViT)
#### 🔹 Introduced
- **Year**: 2020
#### 🔹 Foundations
- Treat images as sequences of patches and apply standard NLP-style Transformers
#### 🔹 Tokenization & Embeddings
- Divide image into **fixed-size patches**
- **Flatten** and **project** each patch into an embedding
- Add **positional embeddings** to maintain spatial order
#### 🔹 Characteristics
- Lacks **inductive bias** like locality or translation invariance (unlike CNNs)
- Relies on **global self-attention** across all patches
- Computational complexity: **Quadratic** in image size → $O(N^2)$
#### 🔹 Comparison with CNNs
- CNNs model **spatial locality**, and learn **low-level features** (e.g., edges, textures)
- CNNs have built-in **translation invariance**
#### 🔹 Summary Issues
- Performs **poorly on midsize datasets**
- Requires **large training datasets**
- Inefficient due to **lack of locality modeling**
### 🧩 Token-to-Token Vision Transformer (T2T-ViT)
#### 🔹 Introduced
- **Year**: 2021
#### 🔹 Motivation
- Address ViT's weakness: ViT needs large datasets and lacks locality modeling
#### 🔹 Components
- **Token-to-token (T2T) module**
- **Transformer encoder**
#### 🔹 Enhancements
- Uses **soft split** and **token aggregation** to capture local structure
- **Progressive refinement** of token representations improves semantic richness
- **Fewer parameters** needed, works better on midsize datasets
#### 🔹 Summary Benefit
- Encodes **local structure**
- Requires **fewer parameters**
- Outperforms CNNs on midsize data
### 🌀 Swin Transformer
#### 🔹 Introduced
- **Year**: 2021
#### 🔹 Key Ideas
- Applies self-attention within **local non-overlapping windows** to reduce computation
- Alternates between **regular** and **shifted** windows to allow cross-window communication
- Builds **hierarchical feature maps** similar to CNNs
#### 🔹 Architecture Techniques
- **Patch merging** reduces the number of tokens as network deepens
- **Shifted window partitioning** enables larger receptive fields efficiently
#### 🔹 Output
- CNN-like **hierarchical representations** with **global context**
#### 🔹 Summary
- Efficient transformer for vision
- Combines **locality** with **global modeling**
- Inspired by **CNN structure**, but keeps transformer flexibility
### 🧱 Masked Autoencoders (MAE)
#### 🔹 Introduced
- **Year**: 2021
#### 🔹 Task
- Reconstruct images from sparsely visible patches (after random masking)
#### 🔹 Why It Works
- High masking forces model to learn **global semantic structure**
- More difficult than inpainting — only a small portion of the image is visible
#### 🔹 Architecture
- **Encoder**: Operates on visible patches only
- **Decoder**: Takes **visible encoded tokens** and **learnable mask tokens**
- **Mask tokens** share a **learnable vector**, with **positional embeddings**
### 🔘 Segment Anything Model (SAM)
#### 🔹 Introduced
- **Year**: 2023
#### 🔹 Architecture
- **Image encoder**: Vision Transformer (ViT)
- **Prompt encoder**: Transformer-based
- **Mask decoder**: Promptable for flexible segmentation based on input cues
### 🧠 Panoptic Segmentation
#### 🔹 Goal
- Unify **semantic segmentation** and **instance segmentation**
#### 🔹 Function
- Assign a **class label** and **instance ID** to every pixel
- Handles "thing" categories (e.g., people, cars) using **instance segmentation**