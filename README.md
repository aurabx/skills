# Aurabox Skills

[![Install in Automatic](https://tryautomatic.app/badges/install.svg)](automatic://install?repo=aurabx/skills)

Skills for working with the [Aurabox](https://aurabox.cloud) medical imaging platform and medical imaging data.

---

<div align="center">

### 🛠️ This project was built by [Aurabox](https://github.com/aurabx)

*Some of our other projects...*

[![Runbeam](https://img.shields.io/badge/Runbeam-Data_Mesh_Platform-purple?style=flat-square)](https://runbeam.io)
[![JMIX](https://img.shields.io/badge/Aurabox-Medical_Imaging_Exchange-blue?style=flat-square)](https://aurabox.cloud)

</div>

---

## Skills

### Aurabox API

| Skill | Description |
|-------|-------------|
| [aurabox-rest-api](skills/aurabox-rest-api/) | Client code for the Aurabox REST API (patients, cases, studies) |

### Medical Imaging

| Skill | Description |
|-------|-------------|
| [dicom-processing](skills/dicom-processing/) | Read, write, and manipulate DICOM files with pydicom and DCMTK |
| [dicomweb-protocol](skills/dicomweb-protocol/) | Build DICOMweb clients (WADO-RS, STOW-RS, QIDO-RS) |
| [medical-imaging-pipelines](skills/medical-imaging-pipelines/) | Format conversion, preprocessing, and ML dataset preparation |

## Installing

### With Automatic

Aurabox skills supports one-click install using Automatic

[![Install in Automatic](https://tryautomatic.app/badges/install.svg)](automatic://install?repo=aurabx/skills)

### Manual

Copy the skills you want into your project's `.claude/skills/` directory or your personal `~/.claude/skills/` directory.

```bash
# Install to a project
cp -r dicom-processing /path/to/your/project/.claude/skills/

# Install globally (available in all projects)
cp -r dicom-processing ~/.claude/skills/

# Install all skills globally
for skill in aurabox-rest-api dicom-processing dicomweb-protocol medical-imaging-pipelines; do
  cp -r "$skill" ~/.claude/skills/
done
```

Restart your agent after copying. Skills trigger automatically based on your prompts.

## Skill details

### aurabox-rest-api

For developers integrating with the Aurabox platform programmatically.

- Full API reference for patients, cases (de-identified patients), and studies
- Working client code in Python, TypeScript, and curl
- Authentication, pagination, and error handling
- Data models with field types and validation rules

**Depends on**: An Aurabox account and API key

### dicom-processing

For anyone writing code that reads or manipulates DICOM files.

- pydicom fundamentals: reading, writing, modifying DICOM datasets
- Tag reference table with common tags, VRs, and data types
- Pixel data access, windowing, and display
- Transfer syntax handling and decompression
- Bulk operations and metadata extraction
- DCMTK command-line tools

**Depends on**: `pip install pydicom` (core), optional `numpy`, `pillow`, `pylibjpeg`

### dicomweb-protocol

For developers building HTTP-based imaging integrations.

- QIDO-RS: searching for studies, series, and instances
- WADO-RS: retrieving DICOM instances, metadata, and rendered images
- STOW-RS: uploading DICOM via multipart MIME (the hard part)
- Complete DICOMwebClient class
- DICOM JSON parsing helpers
- Server-specific notes for Aurabox, Orthanc, dcm4chee, Google Cloud Healthcare

**Depends on**: `pip install requests` (Python examples)

### medical-imaging-pipelines

For technical researchers and ML engineers working with imaging datasets.

- DICOM to NIfTI conversion (SimpleITK and nibabel)
- DICOM to PNG/JPEG with windowing
- DICOM to HDF5 for ML datasets
- CT and MRI intensity normalization
- Resampling, cropping, padding
- Metadata manifest generation
- Complete CT research pipeline example
- PyTorch Dataset integration

**Depends on**: `pip install pydicom numpy SimpleITK nibabel pillow` (full set)

## How skills work

Each skill is a `SKILL.md` file with YAML frontmatter (`name` and `description`). An AI coding agent reads the descriptions at startup to decide when to load each skill. When a prompt matches a skill's description, the full content is loaded into context.

- Skills have zero cost when not triggered (only the description is loaded)
- Multiple skills can coexist without bloating context
- Skills trigger automatically -- no manual invocation needed
