# 🧠 AI Resume Screener

An LLM-powered pipeline that automatically parses a batch of resumes, compares each one against a job description, and ranks candidates by fit — built as part of an AI engineering course project.

Instead of a recruiter manually skimming dozens of PDFs, this tool extracts structured data from the job description **and** every resume using an LLM, then asks the LLM to score each candidate against the role and surface the strongest and weakest matches.

---

## ✨ Features

- 📄 **Multi-format resume parsing** — reads both `.pdf` and `.docx` resumes
- 🧩 **Structured extraction** — converts unstructured job descriptions and resumes into clean, validated JSON using [Pydantic](https://docs.pydantic.dev/) schemas
- 🤖 **LLM-driven matching** — uses Groq's `openai/gpt-oss-120b` model to intelligently compare skills, experience, and requirements (not just keyword matching)
- 🏆 **Automatic ranking** — sorts all candidates by match score and reports the top 2 and bottom 2
- 🧠 **Heading-agnostic parsing** — recognizes experience/skills sections even when resumes use different headings ("Work History", "Employment", "Internships", etc.)

---

## ⚙️ How It Works

```
                    ┌─────────────────────┐
                    │   Job Description    │
                    └──────────┬──────────┘
                               │  LLM Call 1
                               ▼
                    ┌─────────────────────┐
                    │  Structured JobD     │ (skills, experience, education)
                    └──────────┬──────────┘
                               │
        ┌──────────────────────┴───────────────────────┐
        │           For each resume in /resumes/         │
        ▼                                                │
┌───────────────────┐                                    │
│  Extract raw text  │ (PDF/DOCX)                         │
└─────────┬──────────┘                                    │
          │ LLM Call 2                                    │
          ▼                                               │
┌───────────────────┐                                     │
│  Structured Resume  │ (skills, experience, education)    │
└─────────┬──────────┘                                     │
          │ LLM Call 3                                     │
          ▼                                                │
┌───────────────────┐                                      │
│   Match Score (%)   │◄─────────────────────────────────┘
│  + missing skills   │
│  + final verdict     │
└─────────┬──────────┘
          ▼
   Sort all candidates
          ▼
  Print Top 2 / Bottom 2
```

1. **Job description → structured data**: The job description is parsed once by the LLM into a `jobD` schema (required skills, preferred skills, minimum experience, education requirements, responsibilities).
2. **Resume text extraction**: Each file in `resumes/` is read using `pypdf` (PDF) or `python-docx` (DOCX).
3. **Resume → structured data**: The raw resume text is parsed by the LLM into a `Resume` schema (name, contact info, skills, experience entries, education, projects, certifications).
4. **Scoring**: The structured job requirements and structured resume are both passed to the LLM, which returns a `MatchResult` — an overall score (0–100) plus supporting details (matching skills, missing skills, experience fit, and a short verdict).
5. **Ranking**: All candidates are sorted by score, and the top 2 and bottom 2 are printed.

---

## 🛠️ Tech Stack

| Component | Tool |
|---|---|
| LLM Provider | [Groq](https://groq.com/) (`openai/gpt-oss-120b`) |
| Data Validation | [Pydantic](https://docs.pydantic.dev/) |
| PDF Parsing | [pypdf](https://pypdf.readthedocs.io/) |
| DOCX Parsing | [python-docx](https://python-docx.readthedocs.io/) |
| Env Management | [python-dotenv](https://pypi.org/project/python-dotenv/) |
| Language | Python 3 |

---

## 📁 Project Structure

```
day5/
├── resume_parser.py     # Main pipeline: extraction, parsing, scoring, ranking
├── resumes/             # Drop candidate resumes here (.pdf / .docx) — not tracked in git
├── pyproject.toml       # Project dependencies
├── .python-version      # Python version pin
├── .env                 # API keys (not tracked in git)
└── README.md
```

---

## 🚀 Setup & Installation

1. **Clone the repo**
   ```bash
   git clone https://github.com/<your-username>/<repo-name>.git
   cd day5
   ```

2. **Install dependencies** (using [uv](https://docs.astral.sh/uv/) or pip)
   ```bash
   uv sync
   # or
   pip install -r requirements.txt
   ```

3. **Set up your environment variables**

   Create a `.env` file in the project root:
   ```
   GROQ_API_KEY=your_groq_api_key_here
   ```

4. **Add resumes**

   Place candidate resumes (`.pdf` or `.docx`) inside the `resumes/` folder.

5. **Run the pipeline**
   ```bash
   python resume_parser.py
   ```

---

## 📊 Sample Output

```
Processing: john_doe.pdf
Score: 82.5

Processing: jane_smith.docx
Score: 91.0

TOP 2 CANDIDATES
Jane Smith - 91.0 %
{'matching_skills': [...], 'missing_skills': [...], 'verdict': '...'}

John Doe - 82.5 %
{'matching_skills': [...], 'missing_skills': [...], 'verdict': '...'}

LOWEST 2 CANDIDATES
...
```

---

## ⚠️ Known Limitations

- The job description is currently hardcoded inside `resume_parser.py` — could be extended to accept a job description as a file input or CLI argument.
- No retry/backoff logic around the Groq API calls beyond a fixed `time.sleep(5)` between requests.
- Resumes with no extractable text (e.g., scanned image-based PDFs) will fail silently, since `read_pdf` only calls `extract_text()`.

## 🔮 Possible Improvements

- Accept the job description via CLI/file instead of hardcoding it
- Add OCR fallback for scanned/image-based resumes
- Export results to a CSV/JSON report instead of just printing to console
- Add proper error handling around malformed LLM JSON responses
- Parallelize resume processing instead of sequential API calls

---

## 📄 License

This project was built for educational purposes as part of an AI engineering course.
