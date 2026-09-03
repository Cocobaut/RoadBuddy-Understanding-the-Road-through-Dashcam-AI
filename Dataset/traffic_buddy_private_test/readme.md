# TrafficBuddy Dataset Structure
``` 
traffic_buddy_private_test --> this folder will be mount to /data in the docker
│── videos/
│   ├── video_1.mp4
│   ├── video_2.mp4
    └── ...
├── private_test.json
├── download_videos.py
├── readme.md
```
## Downloading Videos
Due to the large size of the video files, they are not included directly in the dataset. Instead, you can download them using the provided URLs.


Each video referenced in `private_test.json` can be accessed using the following URL pattern:
```
https://dl-challenge.zalo.ai/2025/TrafficBuddyHZUTAXF/private_test_ADZDEFHAYD/<video_path>
```

Where `<video_path>` corresponds to the `video_path` field in the JSON files.

### Using the Download Script

A convenience script `download_videos.py` is provided to download all videos automatically:
```bash
# Modify the paths to public_test.json and train.json in the script if needed
python download_videos.py
```

Your solution should use videos stored on disk, not access them directly from the URL.

The /data in the docker should have the traffic buddy structure as below:
```
│─ videos/
│     ├── video_1.mp4
│     ├── video_2.mp4
│     └── ...
│─ private_test.json
