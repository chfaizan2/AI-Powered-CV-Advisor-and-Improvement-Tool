import streamlit as st
import pandas as pd
from PyPDF2 import PdfReader
import docx
import google.generativeai as genai
import os

# Function to extract text from pdf
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

# Configure the Generative AI key
GOOGLE_API_KEY = "AIzaSyD7Rnl8Sbpbnoq4kKzVVf5of6MI89V_vts"
genai.configure(api_key=GOOGLE_API_KEY)
model = genai.GenerativeModel('gemini-1.5-flash')

# Function to query the AI model
def query_model(prompt):
    response = model.generate_content(prompt)
    # Print the type and content of the response for debugging
    st.write("Response type:", type(response))
    st.write("Full response:", response)
    return response

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
            response = query_model(prompt)
            # Extract recommendations from the response
            if hasattr(response, 'candidates') and len(response.candidates) > 0:
                content = response.candidates[0].content
                if isinstance(content, dict) and 'parts' in content and len(content['parts']) > 0:
                    recommendations = content['parts'][0]['text']
                else:
                    recommendations = str(content)  # Fallback to string conversion
            else:
                recommendations = str(response)  # Fallback to string conversion

            # Ensure recommendations is a string and split it into bullet points
            if isinstance(recommendations, str):
                recommendations_list = recommendations.split('\n')
                formatted_recommendations = ""
                for recommendation in recommendations_list:
                    if recommendation.strip():
                        formatted_recommendations += f"- {recommendation.strip()}\n"

                # Display Recommendations
                st.subheader("Recommendations")
                st.write(formatted_recommendations)
            else:
                st.error("Recommendations are not in the expected string format.")
        except Exception as e:
            st.error(f"An error occurred while fetching recommendations: {e}")
    else:
        st.error("Please upload a CV and enter a job description.")
