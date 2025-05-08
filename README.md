
Stress is no longer an exception in today’s work environments — it’s the background noise we’ve all learned to live with. Conversations around burnout, mental fatigue, and declining health have become common, and organizations are increasingly seeking ways to monitor and improve workplace well-being.

With the rise of AI, IoT, and wearable technology, the possibility of detecting stress seamlessly and non-intrusively is closer than ever. Smart wristwatches, in particular, offer a promising gateway into the world of health monitoring, merging convenience with technology in ways never imagined before.

Inspired by this vision, we set out to explore whether stress could be reliably detected using just wristwatch data — no intrusive chest belts, no complex setups. To do this, we turned to the WESAD (Wearable Stress and Affect Detection) dataset from the UC Irvine ML repository, a well-known resource capturing physiological responses during stress, amusement, and baseline states using two devices: the chest-worn RespiBAN and the wrist-worn Empatica E4.

But focusing only on wrist data, as it turned out, wasn’t just a straightforward shortcut — it quickly became a fascinating maze of challenges, surprises, and deep learning.

Check out the Medium article : https://medium.com/@avani.rao/cracking-the-code-of-stress-building-a-wristwatch-only-detection-model-9ecde427b7f5

The Journey: Wrestling with Wristwatch Data

Step 1: 
The Hidden Problem of Sensor Chaos and Feature Engineering
At first glance, the wristwatch data seemed clean: accelerometer readings, skin temperature, electrodermal activity, and blood volume pulse. But there was an immediate hurdle — each sensor operated at a different frequency.

BVP (Blood Volume Pressure) at 64 Hz
ACC (accelerometer for x/y/z axis) 32 Hz
EDA (electrodermal activity) and TEMP (Temperature) at 4 Hz
Stress Labels (Neutral, Baseline, Stress, Amusement, meditation, participant responses with timestamps)
Without alignment, combining these signals for any meaningful modeling would be like trying to choreograph a dance between partners listening to different songs.

Solution:
We resampled all sensor streams to a common 40 Hz frequency. This simple yet crucial step gave me a unified dataset, ready for feature extraction.

Step 2: 
When the Data Fought Back
The signals were too raw. Spikes in EDA and ACC weren’t always due to stress — they could come from random motion.

Solution: 
We applied essential feature engineering:

One-Hot Encoding: Converted categorical emotion labels into a format the model could learn from
Rolling Statistics: Smoothed noisy EDA and TEMP signals using moving averages and standard deviations to reduce random spikes
StandardScaler: Standardized all sensor data to a consistent scale, improving model convergence and fairness

Additional engineered features included HRV from BVP and skin conductance responses from EDA
These transformations gave the data structure. The models started listening to the right signals.

Step 3: 
The Mirage of High Accuracy

With the data resampled, we built a baseline model using standard Random Forest, XGBoost, and Attention LSTM models. The result was thrilling at first: extremely high training accuracy.
And yet, when we ran the model on new subjects, the evaluation accuracy plummeted. The model wasn’t learning the physiology of stress — it was memorizing quirks in the training data.

Classic overfitting.

Step 4: 
Handling Overfitting
Part A: 
We adopted Leave-One-Subject-Out Cross-Validation (LOSO). Each subject was treated as a separate validation fold, forcing the model to generalize to unseen individuals.

The cost? Our sparkling accuracy dropped — drastically. But it was honest.

Realization:
Simple train-test splits (like 80/20) weren’t enough for physiological data. In stress detection, each human is practically a separate “domain” — and a model that generalizes must treat subjects independently.


Part B: 
Handling Imbalance of class with SMOTE
The new labels were still imbalanced, with stress samples underrepresented (as shown in the label distribution chart). To fix this, we used SMOTE (Synthetic Minority Over-sampling Technique) to generate synthetic stress samples and balance the dataset.


Step 5: 
Literature Survey
The literature review supported this challenge, showing that binary classification tends to perform better than multi-class setups in physiological stress detection, especially when emotion classes are overlapping or ambiguous.

Solution:
We simplified the classification task. Instead of forcing the model to predict four emotional states (neutral, baseline, stress, amusement), we collapsed them into:

Multi-Class: Neutral, Baseline, Stress, Amusement
Binary-Class: Stress vs. Non-Stress
Simplifying the targets led to significant gains in model performance.

RESULTS

Binary Random Forest gave the best trade-off — strong average accuracy (0.73), good F1 (0.64), and high stress recall (0.77). Multi-class Random Forest had the highest stress recall (0.82), but slightly lower F1 and precision. LSTM performed poorly and was excluded from further consideration

Findings: What the Wristwatch Told Us
After months of wrestling with data, here’s what we found:

Final Takeaway
If your priority is reliable detection of stress, choose Binary Random Forest — it offers balanced, consistent performance across all key metrics.
If your system can tolerate false positives but must catch all stress cases,then use Multi-class Random Forest.For our goals, where recall and real-world usability matter most, we selected Binary Random Forest as the most effective model.

Key Insights:
Feasibility: Detecting stress through wrist-only methods is achievable, but it comes with its challenges.
Variability: Human variability is high, and wrist signals are chaotic.
Generalization: Overfitting poses a significant risk therefore, LOSO cross validation is essential.
Simplicity: Clean, smart features matter more than model complexity.
Traditional ML models like Random Forest still held their ground surprisingly well.
Explore our full project and findings here Github Link .
Conclusion: Wrist-Based Stress Detection — Possible, but Not Easy
This journey taught us that building a stress detection system using only wrist data is not just an engineering problem — it’s a biological one.

The wrist is a noisy, unpredictable window into the body’s emotional state. Sensors react to much more than emotions. Human variability is massive. And yet, with careful preprocessing, smart feature engineering, and the right validation strategies, it is possible to extract meaningful signals from the noise. Wearables like the Empatica E4—and by extension, future smartwatches — can become powerful allies in tracking mental health. But achieving this requires going beyond chasing accuracy numbers; it demands building models that respect the messiness of human physiology.

The future of stress detection is on our wrists — and it’s messier, more fascinating, and more promising than we ever imagined.

— -

Authored by
Avani Rao, Sowmya Bandaluppi , Karthik Iyer
Graduate Researchers, George Washington University

References
[1] American Psychological Association. (2023, March 8). Stress effects on the body. https://www.apa.org/topics/stress/body

[2] Martin Gjoreski, Hristijan Gjoreski, Mitja Luštrek, and Matjaž Gams. 2016. Continuous stress detection using a wrist device: in laboratory and real life. In Proceedings of the 2016 ACM International Joint Conference on Pervasive and Ubiquitous Computing: Adjunct (UbiComp ‘16). Association for Computing Machinery, New York, NY, USA, 1185–1193. https://doi.org/10.1145/2968219.2968306

[3] Philip Schmidt, Attila Reiss, Robert Duerichen, Claus Marberger, and Kristof Van Laerhoven. 2018. Introducing WESAD, a Multimodal Dataset for Wearable Stress and Affect Detection. In Proceedings of the 20th ACM International Conference on Multimodal Interaction (ICMI ‘18). Association for Computing Machinery, New York, NY, USA, 400–408. https://doi.org/10.1145/3242969.3242985

[4] N. Rashid, T. Mortlock and M. A. A. Faruque, “Stress Detection Using Context-Aware Sensor Fusion From Wearable Devices,” in IEEE Internet of Things Journal, vol. 10, no. 16, pp. 14114–14127, 15 Aug.15, 2023, doi: 10.1109/JIOT.2023.3265768. keywords: {Wearable computers;Anxiety disorders;Sensor fusion;Performance evaluation;Human factors;Context modeling;Sensors;Context-aware models;ensemble learning;stress detection;wearable health sensor fusion

[5] WESAD Dataset


