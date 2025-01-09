import os
import csv
import requests
from firecrawl import FirecrawlApp
import json
import re

# Firecrawl
app = FirecrawlApp(api_key=os.getenv('FIRECRAWL_API_KEY'))

# Ollama
ollama_base_url = "http://localhost:11434"  # Ollama API 주소
model_name = "gemma2" 
#model_name = "phi4" 

def generate_with_ollama(prompt, context=[]):
    try:
        response = requests.post(
            f"{ollama_base_url}/api/generate",
            json={
                "model": model_name,
                "prompt": prompt,
                "context": context,
                "stream": False, 
                 "options": {
                    "temperature": 0,
                }
            },
        )
        response.raise_for_status()  # 오류 발생 시 예외 발생

        # Parsing JSON response
        response_data = response.json()
        return response_data["response"], response_data.get("context", [])

    except requests.exceptions.RequestException as e:
        print(f"Error occurred while calling Ollama API: {e}")
        return None, []

# CSV
csv_filename = 'news.csv'

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

    response, _ = generate_with_ollama(prompt)

    if response:
       try:
           # Remove any ```json and ``` markers
           json_string = re.sub(r"```json\s*", "", response, flags=re.IGNORECASE)
           json_string = re.sub(r"\s*```", "", json_string, flags=re.IGNORECASE)

           start_index = json_string.find("[")
           end_index = json_string.rfind("]") + 1

           if start_index != -1 and end_index != -1:
                json_response = json_string[start_index:end_index]
                json_data = json.loads(json_response)
                return json_data
           else:
                print(f"Error: Could not find json array in response {response}")
                return None
       except json.JSONDecodeError as e:
            print(f"Json Parsing Error: {e}, Response: {response}")
            return None
    else:
        print("Error occurred while calling Ollama API.")
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

# 메인 실행 부분
response = app.scrape_url(url='https://www.website.com/news/', params={
    'formats': ['markdown'],
})

if response and 'markdown' in response:
    markdown_content = response['markdown']
    url = response['metadata']['url']
    print(f"Page URL: {url}")
    
    news_info_list = extract_news_info(markdown_content,url)
    
    if news_info_list:
      
        print(f"news info: {news_info_list}")
        write_to_csv(news_info_list, csv_filename)
        print(f"CSV file '{csv_filename}' were stored.")
    else:
        print("No content found in the content.  ")
else:
    print(f"Markdown content not found in crawl results. Response: {response}")
