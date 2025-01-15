# Preparing for Sentinel's deployment in New Zealand

**By Ben Ibbott - 16/12/2024**
(Internship at ConservationXLabs)

![image](https://github.com/user-attachments/assets/39733f72-1050-4e9d-b896-4a66ee26ff80)
*~Image from LILA BC, New Zealand camera trap dataset. An image used to train the model.*

## Background
Currently, 40 Sentinel-powered cameras are being deployed in New Zealand for the purpose of monitoring invasive species. This is one of the largest deployments of Sentinel to date. For Sentinel to effectively identify and track these invasive species, the model requires vast amounts of data and training to work; and this is where I come in!

I'm Ben, a PhD student at Oxford University currently working on a molecular/computational neuroscience project to better understand the basis of memory. I'm actually on the same PhD programme that CXL's lead AI researcher, Dante Wasmuht, started in 2015. Even more serendipitously, he was the former supervisor of a post-doc who co-supervises me during my own PhD - you could say he is my academic grandfather.

---

Sentinel is an AI-powered technology that pairs with camera traps to monitor wildlife in their natural habitats. Unlike traditional camera traps that simply collect large amounts of unstructured data requiring painstaking manual review, Sentinel analyses images and videos in real time. The smart cameras transfer this classified and organised information to a user-friendly interface accessible by local project leads.

Conservation teams can easily filter through the data to identify relevant species, behaviours, or signs of disease, enabling immediate response based on their specific needs. Moreover, the long-term data collected and categorised by Sentinel provides valuable insights into behaviour patterns and population demographics - information that has traditionally been difficult to gather accurately across large areas.

![image](https://github.com/user-attachments/assets/e57287cc-aa18-472c-9f05-b7f855362d18)
*~Visual summary of different products offered in Sentinel project. Image taken from CXL's website*
	https://sentinel.conservationxlabs.com

Sentinel has been successfully deployed in various locations, proving particularly effective in island conservation projects with limited personnel and challenging terrain. Its real-time classification of invasive species locations enables efficient response, addressing a key limitation of traditional camera traps, which were bottlenecked by data collection and processing times. The local Floreana team in the Galápagos reported that Sentinel was incredibly valuable to their operations.

As mentioned, the focus of this work is for the New Zealand deployment. It will take place in the Auckland Islands, a nature reserve and World Heritage site with native species found nowhere else in the world. Unfortunately, the introduction of invasive pest species (wild pigs, feral cats, non-native rodents, and more) has resulted in the local extinction of numerous native animals and has pushed even more into endangered categories. As such, the goal of this deployment is to track and eradicate these invasive animals to allow native populations to be restored.
	More specific details found here - https://www.doc.govt.nz/our-work/restoring-auckland-island/

 ---

## Internship work
So, what was my role in all of this? Well, to start, I did what all people do when they're handed a huge code repository and that found ways, never before imagined to break it! 

After a week of blindly traversing this code jungle, I cleared through enough digital vegetation and started to assist in optimising different aspects of the pipeline. As only small amounts of data had been parsed through this pipeline before, it meant during my week 1 rampage I uncovered various niche scenarios in which the code was not suitable. An example is: corresponding metadata missing/in the wrong format, breaking the code. Some more substantial changes were also made, such as adding functionality (hotkeys for the GUI) to software that is used in the labelling process, to increase efficiency.

After making my way through the pipeline it was time to label. Simply put, I saw LOTS  of images (around 250,000 to be specific) containing animals from New Zealand and if they had the wrong species label on them or were not detected by the MegaDetector algorithm I fixed that. During this labelling process, I also tracked the frequencies of errors to determine which aspects of the detecting and labelling algorithms were the most error-prone.

It was simultaneously fun and infuriating having to learn about tiny details separating animals whether by sex or species to ensure accurate identification. Here are a few examples to highlight this:

**Weasels vs Stoats**

Weasels (left) are smaller and have shorter tails without the black tip that the Stoats (right) have. 
![image](https://github.com/user-attachments/assets/d2b3961d-8f26-4080-9797-783a005bb0cb)

**Male vs Female Chaffinch's**

A male Chaffinch (left) has distinct coloured feathers with a red-rust body and blueish-grey head. In comparison, the female Chaffinch (right) has a more generic feather colouration.
![image](https://github.com/user-attachments/assets/5356b4e2-1f5c-4108-aba6-56cb4198d2a0)

**Thrush vs Pipit vs Dunnock**

These three birds look quite similar, especially in low-resolution camera trap images. The thrush (left) tends to be the largest of the three and has a spotted black breast pattern. The pipit (middle) is typically more slender, with a narrow beak and a distinct repetitive pattern on its wing feathers. The Dunnock (right) is the smallest of the three birds with a plump body and feathers that are more blueish-grey.
![image](https://github.com/user-attachments/assets/e33f199a-7c71-45bc-9dfd-4e363d81b3cc)



**Lastly, the Labelme interface**
![image](https://github.com/user-attachments/assets/14003ea1-4a29-4b3d-bd49-3ad3dcb8a69a)


Lastly, I developed an analysis pipeline in R that serves multiple purposes. It allows easy manipulation of the LILA New Zealand dataset for extracting specific subsets of images used in model training. The pipeline can analyse labelled data to create useful breakdowns - showing total versus labelled images, species distributions, and progress towards labelling targets for each animal. It also includes error checks to ensure labelled image counts accurately match the CSVs used for model training.

The aforementioned analysis pipeline, figures and other species identification documents are available in our public GitHub repository: 🔗 https://github.com/Bibbott0/CXL_internship

## To summarise
The foundation has now been laid for the initial model training, and its performance can soon be evaluated with the New Zealand deployment. As more data is collected, it can be cleaned and labelled to further improve models effectiveness.

Looking ahead, I'm excited to pursue a small-scale UK project as an extension of this internship. I'll be setting up a Camera trap + Sentinel system in Oxford to collect a novel dataset. After labelling this data, I'll train a model specific to Oxford wildlife, with the potential to expand across more of the UK. This will be a great opportunity to be involved in every aspect of Sentinel: from hardware setup and data extraction through CXL's user interface, to data cleaning and labelling, as well as more advanced work on building and improving the neural networks that power Sentinel's models.

![image](https://github.com/user-attachments/assets/7aa76ee7-12d0-4aef-aa7b-771e7fd42d81)

---

## Bonus images!
#1

![image](https://github.com/user-attachments/assets/e554e993-8730-4845-94ac-7111d2143763)

#2 

![image](https://github.com/user-attachments/assets/9bebc811-bf81-4730-8fc5-bece5a761a6e)

#3 

![image](https://github.com/user-attachments/assets/613f81f0-d2c5-4b0e-b08f-edeb1b966e1b)





