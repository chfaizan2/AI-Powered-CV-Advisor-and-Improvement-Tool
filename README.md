import streamlit as st
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

# Function to query the AI model
def query_model(prompt):
    try:
        # Use a known model like text-bison-001
        response = genai.generate_text(model="models/text-bison-001", prompt=prompt)
        
        # Accessing the content from the candidates list
        if response and response.candidates:
            return response.candidates[0]['output']
        else:
            return "No recommendations were generated."
    except Exception as e:
        # Debugging: Print error details
        st.write("Error details:", e)
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

        Please analyze my CV against the job description for an AI and ML engineer.
        Provide feedback on alignment and suggest improvements or changes to enhance my CV.
        """

        # Get recommendation from AI Model
        try:
            recommendations = query_model(prompt)
            
            # Display Recommendations in Markdown
            st.subheader("Recommendations")
            for line in recommendations.split("\n"):
                if line.strip():  # Avoid displaying empty lines
                    st.markdown(f"- {line.strip()}")  # Display each line as a bullet point
                
        except Exception as e:
            st.error(f"An error occurred while fetching recommendations: {e}")
    else:
        st.error("Please upload a CV and enter a job description.")
