# Neisseria-gonorrheae
1. #downloading the data from website, we developed a data fetching script
-from Bio import Entrez
-import os
-import subprocess
-import xml.etree.ElementTree as ET
-import pandas as pd

2. # Email address to use NCBI Entrez
-Entrez.email = "henrykiema47@gmail.com"

3. # Function to fetch study (SRX) accession numbers for a given project
-(PRJNA) accession number
-def fetch_study_accessions(project_accession):
  -  search_result = Entrez.esearch(db="sra", term=project_accession,
-retmax=1000)
 -   search_record = Entrez.read(search_result)
  -  return search_record['IdList']

4. # Function to fetch run (SRR) accession numbers and metadata for a
-given study (SRX) accession number
-def fetch_run_accessions_and_metadata(study_accession):
 -   fetch_result = Entrez.efetch(db="sra", id=study_accession,
-rettype="xml", retmode="xml")
 -   fetch_record = fetch_result.read()
  -  fetch_result.close()

   - run_accessions = []
    -metadata_list = []

    -root = ET.fromstring(fetch_record)
    -for run_info in root.findall(".//RUN"):
     -   run_accession = run_info.get("accession")
      -  run_accessions.append(run_accession)

       - metadata_dict = {
        -    "run_accession": run_accession,
         -   "study_accession": study_accession,
          -  "total_spots": run_info.get("total_spots"),
           - "total_bases": run_info.get("total_bases"),
            -"platform":
-run_info.find(".//Platform/INSTRUMENT_MODEL").text if
-run_info.find(".//Platform/INSTRUMENT_MODEL") is not None else "",
 -           "instrument": run_info.find(".//Instrument").text if
-run_info.find(".//Instrument") is not None else "",
        }
 -       metadata_list.append(metadata_dict)

  -  return run_accessions, metadata_list

5. # Function to download FASTQ data for a given run accession number
-using fasterq-dump
-def download_fastq(run_accession, output_dir="sequences"):
 -   if not os.path.exists(output_dir):
  -      os.makedirs(output_dir)
   - try:
    -    subprocess.run(['fasterq-dump', '--outdir', output_dir,
-'--split-files', run_accession], check=True)
 -       print(f"Downloaded FASTQ files for: {run_accession}")
  -  except subprocess.CalledProcessError:
   -     print(f"Failed to download FASTQ files for: {run_accession}")

6. # Define the project accession numbers
-project_accession_numbers = ["PRJNA481622", "PRJNA590515"]

7. # Fetch study accessions and run metadata for each project
-all_metadata = []
-for project_accession in project_accession_numbers:
   - print(f"Fetching study accessions for project {project_accession}...")
    -study_accessions = fetch_study_accessions(project_accession)
    -print(f"Identified {len(study_accessions)} study accessions for
-project {project_accession}.")

  -  for study_accession in study_accessions:
   -     print(f"Fetching run accessions and metadata for study
-{study_accession}...")
 -       run_accessions, metadata_list =
-fetch_run_accessions_and_metadata(study_accession)
 -       print(f"Identified {len(run_accessions)} run accessions for
-study {study_accession}.")

 -       all_metadata.extend(metadata_list)
  -      for run_accession in run_accessions:
   -         print(f"Downloading FASTQ for run {run_accession}...")
    -        download_fastq(run_accession)

8. # Convert the list to a DataFrame
-metadata_df = pd.DataFrame(all_metadata)

9. # Print the metadata DataFrame
10. #print(metadata_df)

