import google.generativeai as genai

# Google Gemini API
genai.configure(api_key='[YOUR_API_KEY]')

def gemini_chat(prompt):
  model = genai.GenerativeModel(model_name="gemini-1.5-pro") 
  response = model.generate_content(prompt)
  return response.text

# 
user_prompt = "Hi, how are you?"
gemini_response = gemini_chat(user_prompt)
print(f"사용자: {user_prompt}")
print(f"Gemini: {gemini_response}")

import os
import csv
import requests
from firecrawl import FirecrawlApp
import json
import re
import google.generativeai as genai
import time

# Google Gemini API
genai.configure(api_key='[YOUR_API_KEY]')

# Firecrawl
app = FirecrawlApp(api_key='[YOUR_API_KEY]')

# Gemini
gemini_model_name = "gemini-1.5-pro" 

def generate_with_gemini(prompt):
    try:
        model = genai.GenerativeModel(model_name=gemini_model_name)
        response = model.generate_content(prompt)
        return response.text
    except Exception as e:
        print(f"Error occurred : {e}")
        return None

# CSV 
csv_filename = 'news_list.csv'

def extract_news_info(markdown_content, url):
    """Extract news contents from Markdown contents"""
    prompt = f"""
    Extract the News Title, a short summary, the Date (use today's date if not found), and the URL from this content and return them as a **pure JSON array** without any additional text or code blocks. Each object in the array should represent one news item :

    Content:
    {markdown_content}

    URL:
    {url}

    Desired Output Format:
    [
      {{
        "title": "News Article Title",
        "summary": "A brief summary of the news article.",
        "date": "YYYY-MM-DD",
        "url": "https://example.com/article-url"
      }},
      {{
        "title": "Second News Article Title",
        "summary": "A brief summary of the second news article.",
        "date": "YYYY-MM-DD",
        "url": "https://example2.com/article-url"
      }}

    ]
    """

    response_text = generate_with_gemini(prompt)

    if response_text:
       try:
           # Remove any ```json and ``` markers
           json_string = re.sub(r"```json\s*", "", response_text, flags=re.IGNORECASE)
           json_string = re.sub(r"\s*```", "", json_string, flags=re.IGNORECASE)

           start_index = json_string.find("[")
           end_index = json_string.rfind("]") + 1

           if start_index != -1 and end_index != -1:
                json_response = json_string[start_index:end_index]
                json_data = json.loads(json_response)
                return json_data
           else:
                print(f"Error: Could not find json array in response {response_text}")
                return None
       except json.JSONDecodeError as e:
            print(f"Json Parsing Error: {e}, Response: {response_text}")
            return None
    else:
        print("Gemini API call failed.")
        return None

def write_to_csv(data, filename):
    header = ['Title', 'Summary', 'Date', 'URL']
    file_exists = os.path.isfile(filename)

    with open(filename, 'a', newline='', encoding='utf-8') as csvfile:
        writer = csv.DictWriter(csvfile, fieldnames=header)
        if not file_exists:
          writer.writeheader()
        for item in data:
          try:
            writer.writerow({
                'Title': item['title'],
                'Summary': item['summary'],
                'Date': item['date'],
                'URL': item['url']
            })
          except KeyError as e:
            print(f"Missing key error: {e}, skipping data item: {item}")
            continue

# Main
max_retries = 3
retry_delay = 5
for attempt in range(max_retries):
    try:
        response = app.scrape_url(url='https://www.website.com/news/', params={
            'formats': ['markdown'],
        }) 
        break  
    except requests.exceptions.RequestException as e:
        print(f"Trials {attempt + 1} Fails: {e}")
        if attempt < max_retries - 1:
            sleep_time = retry_delay * (2 ** attempt) 
            print(f"After {sleep_time} sec, retry...")
            time.sleep(sleep_time)
        else:
            print("The maximum number of retries has been reached. The URL cannot be scraped.")
            exit()

if response and 'markdown' in response:
    markdown_content = response['markdown']
    url = response['metadata']['url']
    print(f"Page URL: {url}")

    news_info_list = extract_news_info(markdown_content,url)

    if news_info_list:

        print(f"News info: {news_info_list}")
        write_to_csv(news_info_list, csv_filename)
        print(f"CSV file '{csv_filename}' were stored.")
    else:
        print("No content found in the content.  ")
else:
    print(f"Markdown content not found in crawl results. Response: {response}")
