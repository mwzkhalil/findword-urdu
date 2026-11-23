# FinWord

Find a spoken Urdu phrase by typing it in Urdu script or Roman Urdu. For example, type `moong phali` or **مونگ پھلی** to search a set of recordings and play the proposed match. 

[Open the notebook in Google Colab](https://colab.research.google.com/github/mwzkhalil/findword-urdu/blob/main/findword_Urdu.ipynb) · [Read the notebook on GitHub](./findword_Urdu.ipynb) · [View the seed dataset](./findword_seed.csv)

## Try the dataset first

1. Open the Colab notebook and run **sections 1 and 2** on a CPU runtime.
2. Section 1 builds the seed CSV and prints **96 canonical pairs; 8 positive.**
3. Section 2 checks each label against the official FLEURS Urdu development transcript. On the first run it downloads the 143 MB development audio archive and extracts the eight seed recordings.
4. Play the example and listen for **مونگ پھلی**. Its source file is `6976654990101560269.wav`.

For local use, open a terminal in this folder with Python 3.11 or newer. An existing PyTorch environment can be used.

```sh
python -m pip install -r requirements.txt
python -m jupyterlab findword_Urdu.ipynb
```

## First experiment

How often can an existing Urdu recognizer put the correct recording first? This gives the acoustic model a reference point before training.

1. Run sections 1–2 locally so the eight WAV files are in `findword_work/dev/`.
2. Copy [the eight-file transcript template](./zipformer_predictions.template.csv) to `findword_work/zipformer_predictions.csv`. Transcribe each WAV in quality mode with the released Urdu Streaming Zipformer 20M. Put its actual output in the matching `text` field and save as UTF-8 CSV. The template already contains the filenames. [Zipformer inference instructions](https://github.com/mwzkhalil/Urdu-Streaming-ASR#file-transcription)
3. Run **section 3**. It reports tie-adjusted Recall@1 across the eight queries that have a matching recording. Keep the model revision and decoding mode with that first score.

If you use Colab for sections 1–3 and run Zipformer elsewhere, download the eight seed WAVs from the notebook session, then upload the completed predictions CSV into that session's `findword_work/` folder.

## Train and compare

Sections 4–6 train a small character-level CTC model, search its frame scores for each phrase, and compare the results with fuzzy search over its own decoded transcripts. The model uses 64 log-mel bands, a stride-two convolution and three forward GRU layers. The exact parameter count depends on the training alphabet. The default run downloads the public FLEURS Urdu training archive, about 1.22 GB, and writes a local checkpoint. A local Urdu training manifest can also be used.

This initial snapshot includes the training recipe and a pilot dataset. The trained checkpoint, measured accuracy on voice notes and human-verified timing results are future milestones.

## Pilot data

| Item | Count |
|---|---:|
| FLEURS Urdu development recordings | 8 |
| Distinct queries | 12 |
| Canonical query–recording pairs | 96 |
| Present in the published transcript | 8 |
| Absent from the published transcript | 88 |
| Queries absent from all eight recordings | 4 |

The two Roman spellings per query are hand-written starter aliases. Alternate spellings of the same phrase share a label. Empty `start_s` and `end_s` fields are reserved for human annotation. Claims about voice-note search require a larger test on held-out speakers and independently written queries.

## Sources and attribution

- Audio and reference transcripts come from **FLEURS**, Pakistani Urdu configuration `ur_pk`, by Alexis Conneau and colleagues. FLEURS uses CC BY 4.0. This repository adds query selections, Roman aliases and transcript-derived presence labels. The notebook retrieves recordings from the publisher. [FLEURS dataset](https://huggingface.co/datasets/google/fleurs) · [Source metadata](https://huggingface.co/datasets/google/fleurs/blob/main/data/ur_pk/dev.tsv) · [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- The optional Roman spelling lookup accepts the Urdu training lexicon from **Dakshina** by Brian Roark and colleagues, released under CC BY-SA 4.0. [Dakshina dataset](https://github.com/google-research-datasets/dakshina)
- The transcript baseline uses Muhammad Mahwiz Khalil's **Urdu Streaming Zipformer 20M**. [Code](https://github.com/mwzkhalil/Urdu-Streaming-ASR) · [Released weights](https://huggingface.co/mahwizzzz/Urdu-Streaming-Zipformer-20M)
