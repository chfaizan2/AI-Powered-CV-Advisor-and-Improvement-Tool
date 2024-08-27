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

