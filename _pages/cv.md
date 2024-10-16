---
layout: archive
title: "CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

{% include base_path %}

Education
======
**University of Texas at Austin, College of Computer & Data Science** <br />
Master of Data Science, **GPA: 3.96**, Expected Dec. 2024 <br />
Relevant Coursework: Machine Learning, Deep Learning, Algorithms, Data Science for Healthcare

**Cornell University, College of Engineering** <br />
Master of Engineering in Mechanical Engineering, **GPA: 4.08** <br />
Bachelor of Science in Mechanical Engineering, **GPA: 3.66**

Work Experience
======
**SoftInWay**, *Data Science Intern*   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;    June 2024 – August 2024
* Implemented a transformer model to predict aerodynamic and thermal parameters throughout variable length multi-stage turbine, achieving low mean square error with teacher forcing at inference.
* Used gaussian process modeling to predict whether turbine input parameters would result in physically realistic designs, thus reducing the need for extensive and costly failure data.
* Integrated FAISS for efficient and scalable nearest neighbor search to suggest alternative inputs for unrealistic designs, identifying the closest known successful design based on user inputs.
* Developed code to parse fluid dynamics simulation files and construct bidirectional graphs from node and edge data for integration into graph neural networks.

**Sikorsky Aircraft, Lockheed Martin**, *Structural Analyst*   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   January 2022 - August 2023
* Primary role was to evaluate the structural integrity of systems using hand calculations and finite element analysis to inform design iterations and meet design requirements. 

*Project 1: Test Data Review Automation*
* Developed Python code to automate review of physical test data and flag potential measurement issues, leading to the detection of hundreds of previously missed irregularities. 

*Project 2: Sizing Update Automation*
* Developed a Python program to standardize and automate the process of updating sizing information in a simulated aircraft model, with a focus on checking user inputs and writing test cases.
* Estimated to save 1600 hours of manual processes over several design phases while standardizing the sizing update process. 

*Project 3: Helicopter Maneuver Classification*
* Classified groups of helicopter maneuvers based on operational, multi-dimensional time series data through the 18-week Artificial Intelligence and Machine Learning Fundamentals program. 
* Conducted initial pre-processing steps including extracting data from thousands of text files, grouping maneuvers based on names, and performing data imputations. 
* Achieved a 30% absolute increase in classification accuracy over random selection by generating and selecting features to the flatten the time series data, applying and tuning a random forest classifier. 

Project Experience
======
**Multimedia Recommender**   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;    Dec. 2023 – Feb. 2024
* Generated song recommendations for podcast episodes based on similar themes and mood using a pre-trained sentence-BERT model to encode podcast and song descriptions.
* Created text representations of songs using audio features sourced from the Spotify API. Obtained song lyrics from the Genius API, which were then used to summarize the lyrics from the OpenAI API.
* Clustered songs by their audio features using a gaussian mixture model, and interpreted clusters with decision trees.


**Distracted Driver Monitor**, *Embedded System Design (ECE 5725)*&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fall 2021                                                 
* Created and deployed an application on a Raspberry Pi that receives streaming Pi camera footage along with real-time vehicle speed data to monitor and alert drivers when they are distracted with the car in motion. 
* Used OpenCV and DLib to detect a face and extract facial features and determine whether the driver is drowsy, facing away, or looking away from the road. 
* Accessed real-time vehicle speed from OBDII scanner to inform the onboard system whether to sound an alarm

**Cornell Masters Project**, *Graduate Student Researcher*   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   Jan. 2021 – Dec. 2021
* Developing attitude maneuvers for spacecraft without use of propellent and quantified resulting change in the orbital plane using a six degrees of freedom MATLAB Simulink model.
* Validated results by verifying conservation of linear and angular momentum, as well as attaining machine precision by iterating through step sizes and different solvers. 

Skills
======
Languages: Python, R, SQL, MATLAB

Libraries/Frameworks: PyTorch, NumPy, pandas, scikit-learn, tsfresh


