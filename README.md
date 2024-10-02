Есть папка train/, в ней есть подпапки, а в них pdf файлы, необходимо построить цикл и перебрать последовательно все эти pdf файлы.

С каждым pdf файлом нужно сделать следующее:
В папке train/ есть текстовый файл toc.txt, нужно взять содержимое этого файла и разделить на строки, в каждой строке есть три столбца данных, разделенных запятой:
- Первый столбец имя файла с указанием подпапки "подпапка/имя файла" 
- Второй и третий не используются

Необходимо проверить содержится ли подпапка и имя фала среди имен файлов в первом столбце.

Если подпапка и имя файла содержатся в файле toc.txt, то необходимо перейти к следующему файлу.

А иначе с каждым pdf файлом выполняем следующее:
Извлечь картинки из pdf файла и распознать содержащийся на них текст. В этом тексте важно найти заголовки разделов, записать их одну текстовую переменную, с новой строки каждый заголовок. Снова берём pdf файл и на первой странице пишем слово "Оглавление" и ниже вставляем список заголовков, который мы сохранили ранее в переменную, а имеющееся содержимое pdf файла отодвигаем на одну или несколько страниц в зависимости от объёма оглавления. Сохраняем pdf файл в ту же папку под тем же именем, заменяя исходный файл.

Для выполнения вашей задачи мы можем использовать несколько библиотек на Python, таких как `os` для работы с файлами и папками, `PyPDF2` или `pdfplumber` для работы с PDF-файлами, а также `pytesseract` для распознавания текста на изображениях. Мы также используем библиотеку `Pillow` для извлечения изображений из PDF.

Вот примерный алгоритм решения вашей задачи:

1. Пройтись по всем PDF-файлам в подпапках.
2. Прочитать содержимое файла `toc.txt` и собрать имена файлов в виде множества для быстрого поиска.
3. Сравнить путь к текущему PDF с множеством из `toc.txt`.
4. Если файл не содержится в `toc.txt`, извлечь изображения из PDF и распознать текст, чтобы найти заголовки.
5. Создать новое содержимое PDF с заглавной страницей и заголовками.
6. Сохранить измененный PDF-файл, заменив оригинал.

Вот пример кода для этого процесса:

```python
import os
import re
import PyPDF2
import pdfplumber
from pdf2image import convert_from_path
from PIL import Image
import pytesseract
from reportlab.pdfgen import canvas

def extract_section_titles(pdf_path):
    titles = []
    
    # Извлечение изображений из PDF
    images = convert_from_path(pdf_path)
    
    for image in images:
        # Распознавание текста на каждом изображении
        text = pytesseract.image_to_string(image)
        
        # Регулярное выражение для поиска заголовков (можно изменить по необходимости)
        # Предполагаем, что заголовки начинаются с заглавной буквы и находятся на отдельной строке.
        found_titles = re.findall(r'^(?!\d)([A-ZА-Я][^:]*)(?=\n)', text, re.MULTILINE)
        titles.extend(found_titles)

    return titles

def create_pdf_with_contents(original_pdf_path, titles):
    output_pdf_path = original_pdf_path
    
    # Создание PDF с оглавлением
    temp_pdf_path = "temp.pdf"
    c = canvas.Canvas(temp_pdf_path)
    
    # Заголовок
    c.drawString(100, 800, 'Оглавление')
    
    # Добавляем заголовки
    y_position = 750
    for title in titles:
        c.drawString(100, y_position, title)
        y_position -= 20  # Сдвиг вниз
    
    c.save()
    
    # Объединяем оглавление с оригинальным PDF
    with open(output_pdf_path, 'rb') as original_pdf_file:
        with open(temp_pdf_path, 'rb') as toc_pdf_file:
            original_pdf = PyPDF2.PdfFileReader(original_pdf_file)
            toc_pdf = PyPDF2.PdfFileReader(toc_pdf_file)
            
            output_pdf = PyPDF2.PdfFileWriter()
            
            # Сначала добавляем оглавление
            for page in range(toc_pdf.getNumPages()):
                output_pdf.addPage(toc_pdf.getPage(page))
            
            # Затем добавляем оригинальный PDF
            for page in range(original_pdf.getNumPages()):
                output_pdf.addPage(original_pdf.getPage(page))
                
            with open(output_pdf_path, 'wb') as output_file:
                output_pdf.write(output_file)

def main():
    toc_file_path = 'train/toc.txt'
    toc_files = set()
    
    # Чтение содержимого toc.txt
    with open(toc_file_path, 'r') as toc_file:
        for line in toc_file:
            file_info = line.strip().split(',')
            if file_info:  
                toc_files.add(file_info[0].strip())

    # Проход по всем подпапкам и PDF-файлам
    root_dir = 'train/'
    for subdir, _, files in os.walk(root_dir):
        for file in files:
            if file.endswith('.pdf'):
                pdf_path = os.path.join(subdir, file)
                if pdf_path in toc_files:
                    print(f"The file '{pdf_path}' is already in toc.txt. Skipping...")
                    continue
                
                # Извлечение заголовков
                section_titles = extract_section_titles(pdf_path)
                
                if section_titles:
                    create_pdf_with_contents(pdf_path, section_titles)
                    print(f"Updated '{pdf_path}' with content.")

if __name__ == '__main__':
    main()
```

### Описание кода:

1. **Извлечение заголовков**:
   Функция `extract_section_titles` извлекает заголовки из PDF-файла, конвертируя страницы в изображения и затем распознавая текст с помощью OCR.

2. **Создание нового PDF**:
   Функция `create_pdf_with_contents` создает новый PDF файл с заголовками. Сначала записываются заголовки в новый PDF, а затем оригинальный PDF объединяется с этим новым PDF.

3. **Основная функция**:
   В функции `main()` загружаются все имена файлов из `toc.txt`, после чего происходит обход всех файлов в каталоге `train/`. Если файл не содержится в `toc.txt`, извлекаются заголовки, и создается новый PDF с оглавлением.

### Зависимости:
Убедитесь, что у вас установлены следующие библиотеки:
```bash
pip install PyPDF2 pdfplumber pdf2image pytesseract Pillow reportlab
```

### Примечание:
- Обратите внимание, что вам может понадобиться настроить регулярное выражение для извлечения заголовков в зависимости от формата ваших PDF-документов.
- Также убедитесь, что у вас установлен Tesseract OCR и его путь прописан в системе.
