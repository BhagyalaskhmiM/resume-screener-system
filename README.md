# AI Resume Screening System (Python)

Ranks a folder of resumes against a job description using TF-IDF text
similarity (scikit-learn) combined with a skill-keyword overlap score.

## How it works

1. **Extraction** — reads `.pdf` (via `pdfplumber`), `.docx` (via
   `python-docx`), and `.txt` resumes.
2. **Preprocessing** — lowercases text, tokenizes, and strips stopwords.
3. **Text similarity** — vectorizes the job description and every resume
   with `TfidfVectorizer`, then scores each resume with cosine similarity
   against the job description.
4. **Skill matching** — checks both texts against a built-in dictionary of
   ~100 common skills/tools and reports matched vs. missing skills per
   resume, explicitly and readably.
5. **Combined score** = `0.6 * text_similarity + 0.4 * skill_coverage`
   (weights are adjustable via CLI flags).
6. **Ranking** — sorts resumes by combined score and prints a report; can
   also write a CSV.

## Setup

```bash
pip install -r requirements.txt
```

## Usage

```bash
# Job description from a file, resumes from a folder, print report to console
python resume_screener.py --jd sample_data/job_description.txt --resumes sample_data

# Also write a ranked CSV
python resume_screener.py --jd sample_data/job_description.txt --resumes sample_data --output results.csv

# Job description as raw text instead of a file
python resume_screener.py --jd "Looking for a Python backend developer with AWS experience." --resumes ./my_resumes

# Adjust the weighting between text similarity and skill coverage
python resume_screener.py --jd sample_data/job_description.txt --resumes sample_data --text-weight 0.7 --skill-weight 0.3
```

The `sample_data/` folder has a sample job description and three sample
resumes (strong / moderate / weak match) so you can try it immediately.

## Extending this into a real system

This script is deliberately dependency-light so it's easy to read end to
end. Natural next steps if you want to grow it:

- **Semantic matching** — swap or blend TF-IDF with sentence embeddings
  (e.g. `sentence-transformers`, or an embeddings API) to catch synonyms
  like "k8s" vs "Kubernetes" that exact keyword matching misses.
- **Web/API layer** — wrap `score_resumes()` in a Flask/FastAPI endpoint so
  a frontend can upload files and get results back as JSON.
- **Structured resume parsing** — pull out named entities (years of
  experience, degrees, job titles) with spaCy for more targeted scoring
  instead of scoring the whole document as one bag of words.
- **Bias auditing** — if this ever screens real candidates, audit score
  distributions across demographic groups before relying on it, and keep a
  human in the loop — see the note printed at the end of every report.

## Files

```
resume_screener.py          Core library + CLI
requirements.txt            Dependencies
sample_data/
  job_description.txt       Sample JD
  resume_alice_chen.txt     Strong-match sample resume
  resume_bob_martinez.txt   Moderate-match sample resume
  resume_carol_ng.txt       Weak-match sample resume
```
