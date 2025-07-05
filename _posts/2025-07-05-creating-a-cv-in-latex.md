---
layout: post
title: "Creating a CV in LaTeX: Clean, Structured, and Versioned"
excerpt: How I built a modern, professional CV in LaTeX and shared it on GitHub for easy versioning and reuse.
date: 2025-07-05T12:02:00+02:00
comments: true
tags: [latex, cv, productivity, open-source]
image:
  feature: posts/cv-in-latex/cv-in-latex.jpg
  credit: PngTree
  creditlink: https://pngtree.com/free-backgrounds
---

For years, I maintained my CV using various online builders. They got the job done, but there was always a subscription fee. This last time I tried to update my CV through one of them, their payment process got stuck. They withdrew the subscription fee from my card but didn't enable any premium features. That was the last nail on the coffin for me.

This year, I decided to rebuild my CV from scratch using **LaTeX**. The result is clean, easy to maintain, and version-controlled.

## Why LaTeX?

LaTeX is a typesetting system widely used in academia and technical fields. It lets you separate content from presentation, gives you full control over layout and styling, and produces high-quality PDFs.

For me, LaTeX offered a few major benefits:

- Precise control over formatting  
- Clean separation of content and style  
- Easy versioning using Git  
- Great print and PDF output  
- No reliance on visual editors or WYSIWYG templates

I also liked the idea of making the whole project open source, so others could see how it was done and adapt it to their own needs.

## Tools I Used

- **Editor:** [Overleaf](https://overleaf.com) + local editing in VS Code with LaTeX Workshop  
- **Template:** A custom minimalist layout using the `article` class, with optional support for the [AltaCV class](https://github.com/liantze/AltaCV)  
- **Version control:** [GitHub repository](https://github.com/jonathas/your-cv-repo) with full LaTeX source and compiled PDF  
- **Licensing:** Reusable for personal use under a non-commercial license  

## Sample LaTeX Code

Here’s a basic example using the `article` class:

```latex
\documentclass[11pt,a4paper]{article}
\usepackage[utf8]{inputenc}
\usepackage{geometry}
\geometry{margin=1in}

\begin{document}

\section*{Profile}
Senior Software Engineer with 15+ years of experience across diverse technologies and teams.

\section*{Skills}
\begin{itemize}
  \item Backend: Node.js, TypeScript, Java, PHP
  \item DevOps: AWS, Terraform, Docker, GitHub Actions
\end{itemize}

\section*{Experience}
\textbf{Senior Software Engineer} \hfill Jan 2024 -- Present \\
Vacasa – Prague, Czech Republic (Hybrid) \\
Worked on cloud architecture and workflow automation projects supporting large-scale field operations.

\end{document}
```

If you want a more modern layout, I highly recommend [AltaCV](https://github.com/liantze/AltaCV), a great open-source LaTeX CV class. You can get started with:

```bash
git clone https://github.com/liantze/AltaCV.git
```

Or [download the class file directly](https://raw.githubusercontent.com/liantze/AltaCV/master/altacv.cls) and use it when building your CV.

## Structure and Philosophy

My CV is structured to be clean, readable, and compliant with applicant tracking systems (ATS). It includes:

- A brief summary  
- A key skills section  
- Chronological work experience  
- A compact education section  
- Language skills and legal residency info (since I live in the EU)

Everything fits into **two pages**, and updates are now fast and consistent. I no longer have to worry about formatting bugs or version mismatches.

## Links

- 📂 [CV GitHub Repository](https://github.com/jonathas/your-cv-repo)  
- 📄 [Download the PDF](https://jonathas.com/files/Jonathas_Ribeiro_CV.pdf)

## Final Thoughts

LaTeX isn’t the quickest way to build a CV, but it’s one of the most reliable and rewarding. It’s a great fit for developers, researchers, or anyone who wants full control over how their work is presented.

If you’re curious, I encourage you to clone my repo or use it as inspiration for your own LaTeX CV.
