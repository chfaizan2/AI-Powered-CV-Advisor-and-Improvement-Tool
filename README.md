import streamlit as st
import pandas as pd
from PyPDF2 import PdfReader
import docx
import google.generativeai as genai
import os

# Funcion to extract text from pdf
def extract_text_from_pdf(pdf_file):
    text = ""
    reader = PdfReader(pdf_file)
    for page in reader.pages:
        text += page.extract_text() or ""
    return text

# Function to extract text from docx
def extract_text_from_docx(docx_file):
    doc = docx.Document(docx_file)
    text = "\n".join([paragraph.text for paragraph in doc.paragraphs])
    return text

# Configure the Generative Ai key
GOOGLE_API_KEY = "AIzaSyD7Rnl8Sbpbnoq4kKzVVf5of6MI89V_vts"
genai.configure(api_key = GOOGLE_API_KEY )
model = genai.GenerativeModel('gemini-1.5-flash')