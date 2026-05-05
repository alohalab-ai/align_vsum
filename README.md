# align_vsum Release

This repository provides the release metadata and usage documentation for the benchmark datasets used in the paper release. Due to file size limits, the canonical data files are distributed through Google Drive rather than stored directly in GitHub.

## Download

Google Drive folders:

- Processed JSON files:
  - <https://drive.google.com/drive/folders/1NucujRzoOxSUTtkS4IL59WUUR8fRSxFL?usp=share_link>
- Canonical H5 files:
  - <https://drive.google.com/drive/folders/1446xK_4iAIY6gRGv5wSwbrSA7G2oxw9r?usp=share_link>

Release files:

- `data/processed/tvsum_data.json`
- `data/processed/summe_data.json`
- `data/processed/ovp_data.json`
- `data/processed/youtube_data.json`
- `data/processed/videoxum_data.json`
- `data/processed/mrhisum_data.json`
- `data/tvsum/h5/tvsum.h5`
- `data/summe/h5/summe.h5`
- `data/ovp/h5/ovp.h5`
- `data/youtube/h5/youtube.h5`
- `data/videoxum/h5/videoxum.h5`
- `data/mrhisum/h5/mrhisum.h5`

Recommended local layout:

```text
align_vsum/
├── README.md
└── data/
    ├── processed/
    │   ├── tvsum_data.json
    │   ├── summe_data.json
    │   ├── ovp_data.json
    │   ├── youtube_data.json
    │   ├── videoxum_data.json
    │   └── mrhisum_data.json
    ├── tvsum/h5/tvsum.h5
    ├── summe/h5/summe.h5
    ├── ovp/h5/ovp.h5
    ├── youtube/h5/youtube.h5
    ├── videoxum/h5/videoxum.h5
    └── mrhisum/h5/mrhisum.h5
```

## Dataset Overview

| Dataset | Videos | JSON | H5 |
| --- | ---: | ---: | ---: |
| TVSum | 50 | 8.3 MB | 120 MB |
| SumMe | 25 | 2.7 MB | 36 MB |
| OVP | 50 | 1.4 MB | 11 MB |
| YouTube | 40 | 4.2 MB | 33 MB |
| VideoXum | 5300 | 214 MB | 5.2 GB |
| MR.HiSum | 5590 | 361 MB | 2.3 GB |


## JSON Structure

Each `*_data.json` file is a dictionary keyed by video id such as `video_1` or `video_1020`.

Common top-level fields:

- `frames`: sampled frame-level annotations
- `clips`: clip or shot-level summaries and references
- `aligned_full_document`: full aligned textual document for the video
- `aligned_text_summary`: aligned summary text for the video

Dataset-specific optional fields:

- `video_name`: original source video identifier, present in `ovp`, `videoxum`, and `mrhisum`
- `shot_level_gt`: available in `youtube`

### `frames`

Each item in `frames` is a dictionary with:

- `frame_id`: frame index in the original video frame space
- `description`: textual description for the sampled frame
- `is_gt`: whether the frame is marked as a ground-truth salient frame
- `status`: filtered flag retained from the annotation pipeline

### `clips`

Each item in `clips` is a dictionary with:

- `clip_id`: clip or segment id
- `summary`: textual summary of the clip
- `gt`: whether the clip is marked as salient / ground-truth positive
- `frames`: list of referenced frame ids associated with the clip
- `frame_count`: number of frame ids listed in `frames`
- `status`: filtered flag retained from the annotation pipeline

## H5 Structure

Each canonical H5 file stores one group per video, keyed by the same `video_xxx` identifier used in the processed JSON files.

Common H5 datasets per video:

- `change_points`: segment boundaries
- `gtframe`: keyframes
- `gtscore`: frame-level importance scores
- `n_frame_per_seg`: number of frames per segment
- `n_frames`: number of frames in the original video space
- `n_steps`: number of sampled steps used by the benchmark
- `picks`: sampled frame indices in original frame space
- `shot_level_gt`: shot-level ground-truth labels

## How JSON and H5 Relate

The processed JSON files provide the textual layer of the release, while the H5 files provide the sampling, segmentation, and evaluation metadata.

In typical use:

1. Read `data/processed/<dataset>_data.json` for frame descriptions, clip summaries, and aligned text.
2. Read `data/<dataset>/h5/<dataset>.h5` for `picks`, `change_points`, `gtscore`, and other benchmark metadata.
3. Match videos by their top-level key such as `video_1` or `video_1020`.
4. Use `frame_id` in JSON together with `picks` in H5 to connect text annotations to sampled video frames.

## Minimal Usage Example

```python
import json
import h5py

dataset = "tvsum"
video_id = "video_1"

with open(f"data/processed/{dataset}_data.json", "r", encoding="utf-8") as f:
    processed = json.load(f)

video_data = processed[video_id]
frame_descriptions = video_data["frames"]
clip_summaries = video_data["clips"]
aligned_summary = video_data["aligned_text_summary"]

with h5py.File(f"data/{dataset}/h5/{dataset}.h5", "r") as h5f:
    meta = h5f[video_id]
    picks = meta["picks"][()]
    change_points = meta["change_points"][()]
    gtscore = meta["gtscore"][()]

print("sampled frames:", len(frame_descriptions))
print("h5 picks:", len(picks))
print("first frame description:", frame_descriptions[0]["description"])
print("first change point:", change_points[0].tolist())
```

## Usage Notes

- Use the JSON files when you need text documents, frame- or shot-level descriptions, and aligned textual annotations.
- Use the H5 files when you need benchmark sampling metadata, segmentation, and evaluation targets.
- The release is intended for benchmark use and paper reproduction, not for redistributing raw source media.

## Citation

If you use this release in research, please cite the corresponding paper.
