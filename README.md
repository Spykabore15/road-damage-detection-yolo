Road Damage Detection Using YOLO: Project Report
1. Introduction
This project focuses on the development and evaluation of an object detection system for road damage detection, with the goal of deploying the model on Malaysian roads. To address this problem, several versions of the YOLO (You Only Look Once) model—specifically YOLOv8 and YOLOv11—were tested on multiple datasets with different preprocessing and augmentation strategies.
Throughout this report, I describe the datasets explored, the modeling approach, the experiments conducted, the issues encountered, and the lessons learned. The objective is to present a clear and structured analysis of the process leading to a more robust and representative model.
________________________________________
2. Dataset Exploration and Preprocessing
2.1 Initial Dataset: RDD2022
As this was my first real computer vision project, I initially gravitated toward a large and widely used dataset: RDD2022, a multinational road damage dataset containing images from Japan, India, the Czech Republic, Norway, the United States, and China.
After nearly two days of preprocessing, I uploaded the dataset to Roboflow.
The dataset included:
•	38,355 images
•	A heavily imbalanced class distribution
2.2 First Observations
Using such a large and imbalanced dataset presented two challenges:
1.	Training cost and time — A first naïve YOLOv11 training with:
o	150 epochs
o	batch size 16
o	image size 640
o	patience 30
o	LR0 = 0.001, LRF = 0.01
ran for 15 hours on an NVIDIA L4 GPU.
2.	Model behaviour
o	The model began overfitting around epoch 100.
o	The confusion matrix revealed frequent misclassifications caused by confusion with the background, leading to many false positives and false negatives.
o	Despite these issues, the model’s overall performance was decent given the difficulty and imbalance.
After discussing with my supervisor, I realized that YOLO models do not require extremely large datasets. Instead, they benefit more from:
•	well-curated data
•	balanced class distribution
•	representative samples of the target environment
This insight motivated a new approach.
________________________________________
3. Building a More Representative Dataset
3.1 Transition to RDD2024 (Subset)
Since the model will be deployed in Malaysia, it was essential to use images with similar road conditions. I found RDD2024 on Roboflow, which includes additional classes (8 instead of 4). I selected only the subsets from China and India due to their similarities with Malaysian road infrastructure.
This became Dataset V1, composed of:
•	3,198 images
•	Noticeable class imbalance, but more consistent and representative than RDD2022.
3.2 Training on Dataset V1
I trained YOLOv8s and YOLOv11s with the same configuration:
•	100 epochs
•	batch size 16
•	image size 640
•	save period 10
•	patience 10
Each training ran for approximately 1 hour, and the results were surprisingly strong:
•	Training curves were stable
•	Performance significantly improved
•	YOLOv8s and YOLOv11s delivered very similar results
•	Underrepresented classes still had low-confidence predictions, as expected
This stage confirmed that a smaller but more representative dataset can outperform a very large but broad one.
________________________________________
4. Addressing Class Imbalance Through Augmentation
4.1 Motivation for Dataset V2
To tackle the class imbalance, I attempted to augment only the underrepresented classes. Roboflow applies augmentation uniformly across all classes, which is a limitation. Instead of creating a custom augmentation pipeline in the notebook, I pursued a more complex—and ultimately costly—strategy:
1.	Duplicate Dataset V1
2.	Remove all classes except the underrepresented ones
3.	Apply augmentations
4.	Merge the augmented images back with the original dataset
This produced Dataset V2:
•	4,483 images
•	More balanced class distribution
•	One class (“pedestrian crossing blur”) was removed because it remained underrepresented
4.2 Problems Introduced
This approach caused several issues:
1. Duplicated images
Because augmented_data = previous_data + augmented samples,
merging back duplicated ~900 images in the final dataset.
2. Risky augmentation strategy
Some transformations were naive, repetitive, or not aligned with real-world variations. This creates patterns that can bias the model.
3. Reduced diversity
Removing certain classes unintentionally reduced diversity in the remaining dataset, even when aiming to fix imbalance.
________________________________________
5. Training on Dataset V2
Using the same training configuration as before, both YOLOv8s and YOLOv11s were retrained on Dataset V2.
Results
•	Performance metrics were the highest obtained across all experiments
•	YOLOv8 and YOLOv11 again showed similar behavior
•	However: generalization performance was worse than models trained on Dataset V1
Why did generalization drop?
Based on research and supervisor discussions:
•	Dataset V1 contains natural variations
(lighting, camera angles, noise, backgrounds), leading to stronger generalization.
•	Dataset V2 contains many artificial variations, which are less effective:
o	Augmentations may be too similar or repetitive
o	Artificial diversity does not replace real-world variability
o	The model becomes good at detecting patterns of the augmented data—not real data
This is a common issue:
Augmentation is not about increasing dataset size, but increasing meaningful variability.
________________________________________
6. Generalization Test on Malaysian Road Images
To evaluate real-world performance, I took several images of Malaysian roads after rainfall and tested all the trained models.
Outcome
All models performed poorly:
•	No high-confidence predictions
•	Low-confidence predictions were mostly inaccurate
This demonstrated the necessity of fine-tuning with local data.
Even a few real Malaysian samples in the validation set would have improved robustness significantly.
________________________________________
7. Key Takeaways and Lessons Learned
1. More data ≠ better performance
Large datasets are not automatically beneficial.
Representativeness matters far more than sheer size.
2. Start with a thorough project analysis
A better initial understanding would have reduced wasted compute and time.
3. Augmentation must be problem-driven
For example:
•	Struggles with sunlight → add brightness shifts
•	Struggles with angle variations → use rotation/perspective transforms
•	Struggles with motion blur → incorporate motion blur
Generic augmentation can harm generalization.
4. Monitor class distribution carefully
Removing a class may unintentionally create new imbalances.
5. Avoid repetitive or overly mild augmentation pipelines
Thousands of near-identical samples mislead the model into overfitting.
6. Generalization depends on domain coverage
Even stable training curves do not guarantee strong real-world performance if the dataset lacks environmental diversity.
7. Fine-tuning is essential
Always incorporate images from the deployment environment, even a small set.
________________________________________
8. Deployment
All the different models have been deployed on Roboflow.
________________________________________
9. Conclusion
This project provided valuable hands-on experience in dataset selection, preprocessing, augmentation design, and model evaluation within the YOLO ecosystem. I learned that building an effective computer vision model involves far more than running training sessions—it requires understanding the domain, curating representative data, controlling augmentations carefully, and continuously validating generalization performance.
Future work will involve:
•	Collecting a dedicated Malaysian road dataset
•	Applying targeted augmentations based on observed model weaknesses
•	Fine-tuning YOLO models with local conditions
•	Exploring quantization or optimization for real-time deployment
This iterative approach should lead to a more robust and reliable road-damage detection model suited for Malaysian road conditions.

10. GitHub repository
Spykabore15/road-damage-detection-yolo: Road damage detection system using YOLOv11
