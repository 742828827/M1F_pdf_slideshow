import os
import fitz  # PyMuPDF
import tkinter as tk
from PIL import Image, ImageTk

# PDFファイルのあるフォルダ
pdf_folder = "/home/iharamfg/Desktop/PDFlist"  # あなたのフォルダに変更

# フォルダ内の全ての.pdfファイルを取得
pdf_files = [os.path.join(pdf_folder, file) for file in os.listdir(pdf_folder) if file.endswith(".pdf")]
current_pdf = 0

# PDFのページを画像に変換し、画面にフィットさせる
def get_pdf_page_as_image(pdf_file, page_number=0):
    doc = fitz.open(pdf_file)
    page = doc.load_page(page_number)
    # 高解像度で画像を生成
    pix = page.get_pixmap(matrix=fitz.Matrix(2, 2))  # 解像度を2xに設定
    image = Image.frombytes("RGB", [pix.width, pix.height], pix.samples)
    
    # 画面のサイズを取得
    screen_width = root.winfo_screenwidth()
    screen_height = root.winfo_screenheight()
    
    # 画像のサイズと画面のサイズを比べてリサイズの比率を決定
    img_width, img_height = image.size
    img_aspect_ratio = img_width / img_height
    screen_aspect_ratio = screen_width / screen_height
    
    if img_aspect_ratio > screen_aspect_ratio:
        # 画像が画面よりも横に広い場合
        new_width = screen_width
        new_height = int(screen_width / img_aspect_ratio)
    else:
        # 画像が画面よりも縦に広い場合
        new_width = int(screen_height * img_aspect_ratio)
        new_height = screen_height
    
    # 画像をリサイズ
    image = image.resize((new_width, new_height), Image.Resampling.LANCZOS)
    
    # 画像を画面の中心に配置
    result = Image.new("RGB", (screen_width, screen_height), (255, 255, 255))  # 背景色を白に設定
    result.paste(image, ((screen_width - new_width) // 2, (screen_height - new_height) // 2))
    
    return result

# 次のPDFを表示
def show_next_pdf():
    global current_pdf
    if pdf_files:  # PDFファイルがあるか確認
        current_pdf = (current_pdf + 1) % len(pdf_files)
        image = get_pdf_page_as_image(pdf_files[current_pdf])
        photo = ImageTk.PhotoImage(image)
        label.config(image=photo)
        label.image = photo
        root.after(6000, show_next_pdf)  # 6秒後に次のPDFへ
    else:
        label.config(text="PDFファイルが見つかりません")

# Tkinterの初期設定
root = tk.Tk()
root.attributes("-fullscreen", True)  # 全画面表示
root.config(cursor="none")  # カーソルを非表示にする
root.bind("<Escape>", lambda e: root.quit())  # Escapeキーで終了できるように

label = tk.Label(root)
label.pack()

# 最初のPDFを表示
show_next_pdf()

# Tkinterのイベントループを開始
root.mainloop()