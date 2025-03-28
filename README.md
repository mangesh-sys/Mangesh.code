# Mangesh.code
import streamlit as st
import pdfplumber
from docx import Document
import os
import re

# Define a predefined skill set (expandable)
SKILL_SET = {"python", "java", "c++", "flask", "django", "react", "sql", "mongodb",
             "pandas", "numpy", "scikit-learn", "tensorflow", "html", "css", "javascript"}

def extract_text(file):
    """Extract text from PDF or DOCX file"""
    ext = os.path.splitext(file.name)[1].lower()
    
    if ext == ".pdf":
        return extract_text_from_pdf(file)
    elif ext == ".docx":
        return extract_text_from_docx(file)
    else:
        return "Unsupported file format!"

def extract_text_from_pdf(file):
    """Extracts text from PDF file"""
    text = ""
    with pdfplumber.open(file) as pdf:
        for page in pdf.pages:
            if page.extract_text():
                text += page.extract_text() + "\n"
    return text

def extract_text_from_docx(file):
    """Extracts text from DOCX file"""
    doc = Document(file)
    text = "\n".join([para.text for para in doc.paragraphs])
    return text

def clean_text(text):
    """Cleans extracted text"""
    text = re.sub(r'\n+', '\n', text)  # Remove extra new lines
    text = re.sub(r'\s+', ' ', text).strip()  # Remove extra spaces
    text = re.sub(r'[^a-zA-Z0-9,.\-@ ]', '', text)  # Remove special characters
    return text

def extract_skills(text):
    """Extracts skills from resume text"""
    words = set(re.findall(r'\b\w+\b', text.lower()))
    return list(SKILL_SET.intersection(words))

def calculate_match_percentage(resume_skills, job_description):
    """Compare resume skills with job description and calculate match percentage"""
    job_words = set(re.findall(r'\b\w+\b', job_description.lower()))
    matched_skills = resume_skills.intersection(job_words)
    match_percentage = (len(matched_skills) / len(resume_skills)) * 100 if resume_skills else 0
    return round(match_percentage, 2), matched_skills

# Streamlit GUI
st.set_page_config(page_title="Resume Analyzer", page_icon="📄", layout="wide")

# Custom CSS for Styling
st.markdown("""
    <style>
    body {
        font-family: 'Arial', sans-serif;
        background-color: #f4f4f9;
        margin: 0;
        padding: 0;
    }
    header {
        background-color: #4CAF50;
        padding: 20px;
        text-align: center;
        color: white;
    }
    .container {
        padding: 20px;
    }
    .stTextArea textarea {
        font-size: 16px !important;
    }
    .stButton > button {
        background-color: #4CAF50;
        color: white;
        font-size: 18px;
    }
    .stProgress .st-bo {
        background-color: #4CAF50 !important;
    }
    </style>
""", unsafe_allow_html=True)

# Header
st.markdown('<header><h1>Resume Analyzer</h1><p>Upload your resume and compare it with job descriptions!</p></header>', unsafe_allow_html=True)

# Main Content
with st.container():
    # File uploader for resume
    uploaded_file = st.file_uploader("Upload Resume (PDF/DOCX)", type=["pdf", "docx"])
    # Text area for job description
    job_description = st.text_area("Paste Job Description Here")

    if uploaded_file and job_description:
        # Extract and clean text from resume
        resume_text = extract_text(uploaded_file)
        cleaned_text = clean_text(resume_text)
        # Extract skills from resume
        extracted_skills = set(extract_skills(cleaned_text))
        # Calculate match percentage
        match_percent, matched_skills = calculate_match_percentage(extracted_skills, job_description)

        # Display extracted resume text
        st.subheader("📜 Extracted Resume Text:")
        st.text_area("Resume Content", cleaned_text, height=200)

        # Display extracted skills
        st.subheader("🎯 Extracted Skills:")
        st.write(extracted_skills)

        # Display job matching score
        st.subheader("✅ Job Matching Score:")
        st.progress(match_percent / 100)  # Progress bar
        st.write(f"**Match Percentage:** {match_percent}%")
        st.write(f"**Matched Skills:** {matched_skills}")
