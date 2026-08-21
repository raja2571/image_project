Image-Based Entity Value Extraction Using EasyOCR


This project implements an image-based entity value extraction system using Optical Character Recognition (OCR).

The primary objective is to process product images, extract text embedded in those images, identify the required entity/value from the extracted text, and normalize the detected unit into a standardized format.

The project is implemented in Python using Google Colab, with images and datasets stored in Google Drive.

The pipeline uses EasyOCR for text detection and recognition, followed by custom Python logic based on regular expressions (Regex) and predefined unit mappings to identify entity-specific values such as:

Width
Depth
Height
Item weight
Maximum weight recommendation
Voltage
Wattage
Item volume

The current implementation is designed to handle a large image collection, including approximately 10,000 images, and incorporates batch processing and checkpoint/resume functionality so that processing can continue from the point where it was interrupted.

🎯 Problem Statement
Main Problem Statement

Product information is frequently available only in the form of images. Important attributes such as dimensions, weight, voltage, wattage, and volume may appear as printed or embedded text inside product images.

Manually extracting these values from thousands of product images is:

Time-consuming
Error-prone
Difficult to scale
Inconsistent
Expensive when performed manually

Therefore, the problem is to develop an automated system that can:

Read product images.
Detect and recognize text present in the images.
Extract the relevant entity/value from the OCR output.
Identify the numerical value and its corresponding unit.
Normalize unit abbreviations into standard unit names.
Store the extracted information in a structured CSV dataset.
Process a large number of images efficiently.
Preserve processing progress so that the system can resume after interruption.
🧩 Specific Problem Statements
Problem 1 — Image Text Extraction

Given a product image, automatically identify and extract the text appearing in the image.

Solution:
EasyOCR is used to perform Optical Character Recognition.

Problem 2 — Entity-Specific Value Extraction

OCR can extract a large amount of text from an image, but the project requires a specific entity value.

For example:

Input OCR text:
"Product dimensions 20 cm x 15 cm x 10 cm"

Required entity:
width

Expected extraction:
20 centimeter


The project therefore searches OCR text for a numerical value followed by an appropriate unit.

Problem 3 — Unit Recognition and Normalization

Different products may represent the same unit using different abbreviations.

For example:

cm  → centimeter
m   → meter
mm  → millimeter
kg  → kilogram
g   → gram
V   → volt
W   → watt


The system maps recognized abbreviations to standardized unit names using a predefined units_dict.

Problem 4 — Large-Scale Image Processing

Processing thousands of images in a single uninterrupted operation can be computationally expensive and vulnerable to interruptions.

The project therefore divides processing into batches.

Current implementation:

BATCH_SIZE = 100


After each batch, the processed results are saved to Google Drive.

Problem 5 — Resume Processing After Interruption

If Google Colab disconnects or the processing is interrupted, restarting from image 1 would waste significant processing time.

The project checks whether an existing progress CSV is available and resumes processing from unprocessed records.

This provides a checkpoint-based processing mechanism.

🏗️ System Architecture

The overall architecture can be represented as:

                    ┌───────────────────────┐
                    │   Product Image Data  │
                    │     ~10K Images       │
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │     Google Drive      │
                    │ Image Storage + CSV   │
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │     Google Colab      │
                    │     Python Runtime    │
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │       EasyOCR         │
                    │ Text Detection +      │
                    │ Text Recognition      │
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │     Raw OCR Text      │
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │    Text Cleaning      │
                    │ Regex / Normalization  │
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │ Entity Value Matcher  │
                    │ Entity + Number + Unit│
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │   Unit Normalization  │
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │ Structured CSV Output │
                    │ raw_text              │
                    │ cleaned_text          │
                    │ entity_value          │
                    └───────────────────────┘

🔄 End-to-End Workflow

The complete workflow is:

Dataset
   ↓
Image URL / Image Filename
   ↓
Locate Image in Google Drive
   ↓
Load Image
   ↓
EasyOCR
   ↓
Raw OCR Text
   ↓
Text Cleaning
   ↓
Identify Required Entity
   ↓
Regex-Based Number + Unit Detection
   ↓
Unit Normalization
   ↓
Store Result
   ↓
Save Batch
   ↓
Continue Until Dataset Completion

🧠 Architecture Components
1. Data Layer

The project uses:

Product image files
CSV dataset
Image URLs
Entity names

The notebook references:

dataset2.csv


and an image directory:

downloaded_images


Both are stored in Google Drive.

2. OCR Layer

The OCR component is implemented using:

easyocr.Reader(['en'])


EasyOCR is responsible for:

Detecting text regions
Recognizing English text
Returning detected text from the image

The notebook initializes the EasyOCR English reader before processing the images.

3. Text Processing Layer

After OCR, all detected text segments are combined:

raw_text = " ".join(results)


A cleaned version is then generated by removing unwanted characters:

cleaned_text = re.sub(
    r'[^a-zA-Z0-9.\s]',
    '',
    raw_text
)


Therefore, the system maintains both:

Raw OCR Text
      +
Cleaned OCR Text


This is useful for debugging and downstream processing.

🔍 Entity Extraction Architecture

The core extraction function is:

extract_value_with_unit(text, entity_name)


The function first verifies that:

OCR text exists.
The requested entity exists in the unit dictionary.

It then obtains the list of valid units for that entity.

A regular expression searches for:

NUMBER + UNIT


Examples:

120 V
120V
12.5 kg
20 cm
5 L


The extracted abbreviation is then converted into its complete unit name.

For example:

120 V


becomes:

120 volt

📏 Supported Entity Types

The current implementation defines unit mappings for the following entities:

Entity	Supported Units
Width	centimeter, meter, millimeter, inch, foot, yard
Depth	centimeter, meter, millimeter, inch, foot, yard
Height	centimeter, meter, millimeter, inch, foot, yard
Item Weight	gram, kilogram, microgram, milligram, ounce, pound, ton
Maximum Weight Recommendation	gram, kilogram, microgram, milligram, ounce, pound, ton
Voltage	kilovolt, millivolt, volt
Wattage	kilowatt, watt
Item Volume	liter, milliliter, centiliter, deciliter, microliter, cubic foot, cubic inch, cup, fluid ounce, gallon, imperial gallon, pint, quart

These mappings are explicitly implemented in the notebook's units_dict.

🧹 Text Cleaning

The system maintains two forms of OCR text.

Raw Text

The original OCR output:

raw_text

Cleaned Text

Special/non-alphanumeric characters are removed while retaining:

Alphabetic characters
Numbers
Decimal points
Spaces
cleaned_text = re.sub(
    r'[^a-zA-Z0-9.\s]',
    '',
    raw_text
)


The raw text is preserved because it can be useful when investigating OCR errors.

⚙️ Image Processing Pipeline

Each image is processed using:

process_image(image_path, entity_name)


The function performs three major operations:

Step 1 — File Validation

The system first checks whether the image exists.

If the image is missing:

File Not Found


is returned.

Step 2 — OCR

EasyOCR reads the image:

results = reader.readtext(image_path, detail=0)

Step 3 — Entity Extraction

The OCR result is passed to the entity extraction function:

entity_value = extract_value_with_unit(
    raw_text,
    entity_name
)


The function returns:

raw_text
cleaned_text
entity_value


This processing logic is implemented directly in the notebook.

📦 Batch Processing Architecture

Because the project is designed for a large number of images, processing is divided into batches.

The configured batch size is:

BATCH_SIZE = 100


Therefore:

Images 1–100
      ↓
Save Progress

Images 101–200
      ↓
Save Progress

Images 201–300
      ↓
Save Progress

...


This prevents the entire dataset from depending on one long-running execution.

💾 Checkpoint and Resume Mechanism

The project contains a resume mechanism.

The output file is:

final_output_batches.csv


stored in Google Drive.

Before starting, the system checks whether the progress file already exists.

If it exists:

Existing progress
       ↓
Load previous CSV
       ↓
Update current DataFrame
       ↓
Find first unprocessed row
       ↓
Resume processing


This is particularly useful in Google Colab because runtime sessions can disconnect.

The notebook explicitly implements this checkpoint mechanism.

🗂️ Input Data

The project expects a CSV containing information such as:

image_link
entity_name


The image filename is extracted from the image URL:

img_filename = row['image_link'].split('/')[-1]


The filename is then combined with the configured image directory:

img_full_path = os.path.join(
    IMAGE_FOLDER_PATH,
    img_filename
)


This allows the CSV metadata to be connected to the locally stored image.

📤 Output Data

The processing pipeline adds the following columns:

Column	Description
raw_text	Complete OCR text detected from the image
cleaned_text	Cleaned OCR text
entity_value	Extracted numerical value with normalized unit

Example:

image_link:
.../product123.jpg

entity_name:
voltage

raw_text:
Input 120V AC

cleaned_text:
Input 120V AC

entity_value:
120 volt

🛠️ Technologies Used
Programming Language

Python

Python is used for:

Data processing
Image processing
OCR
Regular expressions
CSV manipulation
File management
Development Environment

Google Colab

The project was implemented in Google Colab.

The notebook uses the Python 3 kernel and is configured to use GPU acceleration.

OCR Framework

EasyOCR

Used for:

Text detection
Text recognition
Image-to-text conversion

The notebook installs EasyOCR and initializes an English OCR reader.

Data Processing

Pandas

Used for:

Reading CSV files
Managing datasets
Adding columns
Updating rows
Saving processed data
Image Processing

OpenCV

The project installs and imports:

cv2


alongside EasyOCR and Pandas.

Regular Expressions

Python's:

re


module is used for:

Text cleaning
Number detection
Unit detection
Entity-value extraction
Storage

Google Drive

Google Drive is used to store:

Input images
Input CSV datasets
Intermediate processing data
Final output CSV

The notebook mounts Google Drive using:

drive.mount('/content/drive')


📚 Libraries

The primary libraries used are:

os
re
pandas
easyocr
opencv-python-headless
google.colab


Installation:

pip install easyocr pandas opencv-python-headless


This installation command is present in the notebook.

🖥️ Google Colab Implementation
Step 1 — Open Google Colab

Create or open a Google Colab notebook.

Step 2 — Enable GPU

The notebook is configured with GPU acceleration.

A GPU runtime is useful because OCR over thousands of images can be computationally intensive.

Step 3 — Install Dependencies

Run:

!pip install easyocr pandas opencv-python-headless

Step 4 — Import Libraries
import os
import re
import pandas as pd
import easyocr
import cv2
from google.colab import drive

Step 5 — Mount Google Drive
drive.mount('/content/drive')


Authorize Google Colab to access the required Google Drive files.

Step 6 — Configure Paths

Update:

IMAGE_FOLDER_PATH = '/content/drive/MyDrive/Raja Project/downloaded_images'

CSV_FILE_PATH = '/content/drive/MyDrive/Raja Project/dataset2.csv'


The paths should be changed according to the user's Google Drive structure.

Step 7 — Initialize OCR
reader = easyocr.Reader(['en'])


The first execution may download EasyOCR's detection and recognition models.

Step 8 — Start Batch Processing

The notebook reads the CSV, creates the required output columns, processes images in batches, and saves progress after each batch.
▶️ How to Run
Option 1 — Google Colab
Open the notebook in Google Colab.
Enable GPU runtime.
Mount Google Drive.
Upload/store the image dataset.
Upload/store the input CSV.
Update the paths in the notebook.
Install dependencies.
Initialize EasyOCR.
Run the processing cells.
Wait for batch processing to complete.
Retrieve the generated CSV from Google Drive.
📋 Quick Start
pip install easyocr pandas opencv-python-headless


Then:

from google.colab import drive

drive.mount('/content/drive')


Initialize OCR:

import easyocr

reader = easyocr.Reader(['en'])


Configure:

IMAGE_FOLDER_PATH = '/content/drive/MyDrive/Raja Project/downloaded_images'
CSV_FILE_PATH = '/content/drive/MyDrive/Raja Project/dataset2.csv'


Run the batch-processing pipeline.

📦 Output

The primary processed output is:

final_output_batches.csv


The output contains the original dataset information plus:

raw_text
cleaned_text
entity_value


The output is saved to Google Drive so that processing progress is retained.

🧾 Project Summary

This project provides an automated OCR-based solution for extracting structured product attributes from images.

The key architecture is:

Product Images
      ↓
EasyOCR
      ↓
Raw Text
      ↓
Text Cleaning
      ↓
Entity-Specific Regex Matching
      ↓
Unit Normalization
      ↓
Structured CSV


The main strengths of the implementation are:

Automated image text extraction
Entity-specific value extraction
Unit normalization
Large-scale batch processing
Checkpoint-based processing
Resume capability
Google Drive integration
GPU-enabled Google Colab execution
👨‍💻 Project Implementation

Environment: Google Colab
Language: Python
OCR: EasyOCR
Data Processing: Pandas
Image Processing: OpenCV
Pattern Matching: Python Regex
Storage: Google Drive
Input: Product images + CSV metadata
Output: Structured CSV with OCR and extracted entity values
