# FinChart-Bench
The repository includes three folders—`MC_images`, `QA_images`, and `TF_images`—each containing the images corresponding to its respective task. 

Additionally, it provides three JSON files—`MC_data.json`, `QA_data.json`, and `TF_data.json`—each containing the image path, question, and ground truth answer for the associated task.

``` json
# TF
  {
    "image": "1243210261_13_crop_0_q1.jpg",
    "question": "The gross margin percentage decreased from 1Q11 to 1Q15.",
    "answer": "True"
  },
# MC
  {
    "image": "1243210261_13_crop_0_q1.jpg",
    "question": "Which year had the highest gross profit according to the chart?",
    "choices": {
      "A": "1Q11",
      "B": "1Q12",
      "C": "1Q14",
      "D": "1Q15"
    },
    "answer": "C"
  },
# QA
  {
    "image": "1243210261_13_crop_0_q1.jpg",
    "question": "What is the difference in gross profit ($M) between 1Q11 and 1Q15?",
    "reasoning": "From the chart, the gross profit in 1Q11 is approximately $710M, and in 1Q15 it is approximately $850M. The difference is $850M - $710M.",
    "answer": 140
  },
```

## How to test:
``` python
import os
import sys
import json
import argparse
import base64
from PIL import Image, ImageDraw, ImageFont

def acc(predict, ground_truth):
    correct = sum(p == gt for p, gt in zip(predict, ground_truth))
    accuracy = correct / len(ground_truth)
    return accuracy

parser = argparse.ArgumentParser(description="Chart detection by year")
parser.add_argument('--type', type=str, required=True, help='Type of questions to process')

args = parser.parse_args()

types = args.type

test_folder = f"../combined_data"

MC_images = test_folder + f"/MC_images"
QA_images = test_folder + f"/QA_images"
TF_images = test_folder + f"/TF_images"

MC_json = test_folder + f"/MC_data.json"
QA_json = test_folder + f"/QA_data.json"
TF_json = test_folder + f"/TF_data.json"

cache_file = f"results_cache_{types}.json"

# Load existing results if cache exists
if os.path.exists(cache_file):
    with open(cache_file, 'r') as f:
        cached_results = json.load(f)
else:
    cached_results = {}

if types == "MC":
    image_folder = MC_images
    with open(MC_json, "r") as f:
        data_list = json.load(f)
elif types == "QA":
    image_folder = QA_images
    with open(QA_json, "r") as f:
        data_list = json.load(f)
else:
    image_folder = TF_images
    with open(TF_json, "r") as f:
        data_list = json.load(f)
json_file = {item["image"]: item for item in data_list}
    
image_files = [f for f in os.listdir(image_folder) if os.path.isfile(os.path.join(image_folder, f))]

predicts = []
gt = []

for image, item in json_file.items():
    name_part, ext = os.path.splitext(image)
    name_part = name_part.rsplit('_', 1)[0]
    new_image = name_part + ext
    if new_image not in image_files:
        continue

    if image in cached_results:
        response = cached_results[image]["prediction"]
        answer = cached_results[image]["answer"]
        print("Cached:", response)
    else:
        image_path = os.path.join(image_folder, new_image)
        if types == "MC":
            question = item["question"]
            choice = item["choices"]
            answer = item["answer"].lower()
            prompt = f"""Question: {question}\nChoice: {choice}\nOutput correct label only in this format: 'Result = [[ label ]]'"""
            
        elif types == "QA":
            question = item["question"]
            reason = item["reasoning"]
            answer = float(item["answer"])
            prompt = f"""Question: {question}\nOutput correct answer only in this format: 'Result = [[ answer ]]'"""
            
        else:
            question = item["question"]
            answer = item["answer"].lower()
            prompt = f"""Question: {question}\nOutput true or false label only in this format: 'Result = [[ label ]]'"""

        TODO
        #################################################
        # Now you have the image path in `image_path`, test prompt in `prompt`, and ground truth in `answer`.
        # You can now test your own model
        #################################################
        

        print(response)
        print(answer)

        cached_results[image] = {
            "prediction": response,
            "answer": answer
        }

        with open(cache_file, 'w') as f:
            json.dump(cached_results, f, indent=2)
        
    predicts.append(response)
    gt.append(answer)

accuracy = acc(predicts, gt)
print(accuracy)
```
