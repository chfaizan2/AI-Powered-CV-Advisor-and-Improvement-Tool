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

# Functtion to query the AI model
def query_model(prompt):
    response = model.generate_content(prompt)
    # Extracting the generated text from the response
    try:
        return response.candidates[0].content
    except (AttributeError, IndexError):
        return "Unable to retrieve recommendations. Please try again."
        
# Streamlit web application
st.title("CV Advisor")

# File upload
cv_file = st.file_uploader("Upload Your CV (PDF or DOCX)", type=['pdf', 'docx'])
job_description = st.text_area("Enter Job Description")

if st.button("Get Recommendation"):
    if cv_file and job_description:
        # Extract text from the CV
        if cv_file.type == "application/pdf":
            cv_text = extract_text_from_pdf(cv_file)
        elif cv_file.type == "application/vnd.openxmlformats-officedocument.wordprocessingml.document":
            cv_text = extract_text_from_docx(cv_file)

# Create prompt for AI model
        prompt = f"""
        CV:
        {cv_text}

        Job Description:
        {job_description}

        I will provide my CV and a job description. As an expert, please analyze my CV against the job description
        and tell me if it aligns well with the job requirements or not.
        Additionally, please suggest what can be added or removed from my CV, and recommend any improvements.
        """
        # Get recommendation from AI Model
        try:
            recommendations = query_model(prompt)
            # Display Recommendations
            st.subheader("Recommendations")
            st.write(recommendations)
        except Exception as e:
            st.error(f"An error occurred while fetching recommendations: {e}")
    else:
        st.error("Please upload a CV and enter a job description.")
