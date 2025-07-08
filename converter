import fitz
import requests
import json

API_KEY = ""
FOLDER_ID = ""

def pdf_to_text_array(pdf_path):
    text_array = []
    try:
        with fitz.open(pdf_path) as doc:
            for page in doc:
                text_array.append(page.get_text())
    except Exception as e:
        print(f"Ошибка при чтении PDF: {e}")
    return text_array

def extract_resume_json(resume_text_array):
    resume_text = "\n".join(resume_text_array)

    system_prompt = (
        "Ты помощник, который преобразует текст резюме в структурированный JSON. "
        "Верни только JSON следующей структуры (если что-то не найдено — оставь пустым) или добавь в отдельный пункт то, чего нет в списке:\n"
        "{\n"
        "  \"ФИО\": \"\",\n"
        "  \"Опыт_работы\": \"\",\n"
        "  \"Хард_скиллы\": [],\n"
        "  \"Софт_скиллы\": [],\n"
        "  \"Контакты\": \"\",\n"
        "  \"Образование\": \"\",\n"
        "  \"Проекты\": []\n"
        "}\n"
        "Не добавляй объяснений, только JSON!"
    )

    data = {
        "modelUri": f"gpt://{FOLDER_ID}/yandexgpt",
        "completionOptions": {"temperature": 0.3, "maxTokens": 3000},
        "messages": [
            {"role": "system", "text": system_prompt},
            {"role": "user", "text": resume_text}
        ]
    }

    headers = {
        "Authorization": f"Api-Key {API_KEY}",
        "Content-Type": "application/json",
        "Accept": "application/json"
    }

    response = requests.post(
        "https://llm.api.cloud.yandex.net/foundationModels/v1/completion",
        headers=headers,
        json=data
    )

    try:
        result = response.json()
        answer = result["result"]["alternatives"][0]["message"]["text"]

        if answer.strip().startswith("```"):
            answer = answer.strip().strip("`")
            first_newline = answer.find("\n")
            answer = answer[first_newline + 1:]  
            answer = answer.strip()

        parsed_json = json.loads(answer)
        return parsed_json

    except Exception as e:
        print("Ошибка при обработке ответа YandexGPT:", e)
        print("Ответ модели:\n", response.text)
        return None


def save_json_to_file(data, filename="resume.json"):

    with open(filename, 'w', encoding='utf-8') as f:
        json.dump(data, f, ensure_ascii=False, indent=2)
    print(f"JSON сохранён в файл: {filename}")

if __name__ == "__main__":
    pdf_path = "resume.pdf"
    resume_pages = pdf_to_text_array(pdf_path)

    if resume_pages:
        resume_json = extract_resume_json(resume_pages)
        if resume_json:
            save_json_to_file(resume_json)
