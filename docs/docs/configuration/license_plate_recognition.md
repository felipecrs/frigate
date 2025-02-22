---
id: license_plate_recognition
title: License Plate Recognition (LPR)
---

Frigate can recognize license plates on vehicles and automatically add the detected characters as a `sub_label` to objects that are of type `car`. A common use case may be to read the license plates of cars pulling into a driveway or cars passing by on a street with a dedicated LPR camera.

Users running a Frigate+ model (or any custom model that natively detects license plates) should ensure that `license_plate` is added to the [list of objects to track](https://docs.frigate.video/plus/#available-label-types) either globally or for a specific camera. This will improve the accuracy and performance of the LPR model.

Users without a model that detects license plates can still run LPR. A small, CPU inference, YOLOv9 license plate detection model will be used instead. You should _not_ define `license_plate` in your list of objects to track.

LPR is most effective when the vehicle’s license plate is fully visible to the camera. For moving vehicles, Frigate will attempt to read the plate continuously, refining recognition and keeping the most confident result. LPR will not run on stationary vehicles.

## Minimum System Requirements

License plate recognition works by running AI models locally on your system. The models are relatively lightweight and run on your CPU. At least 4GB of RAM is required.

## Configuration

License plate recognition is disabled by default. Enable it in your config file:

```yaml
lpr:
  enabled: True
```

## Advanced Configuration

Fine-tune the LPR feature using these optional parameters:

### Detection

- **`detection_threshold`**: License plate object detection confidence score required before recognition runs.
  - Default: `0.7`
  - Note: If you are using a Frigate+ model and you set the `threshold` in your objects config for `license_plate` higher than this value, recognition will never run. It's best to ensure these values match, or this `detection_threshold` is lower than your object config `threshold`.
- **`min_area`**: Defines the minimum size (in pixels) a license plate must be before recognition runs.
  - Default: `1000` pixels.
  - Depending on the resolution of your cameras, you can increase this value to ignore small or distant plates.

### Recognition

- **`recognition_threshold`**: Recognition confidence score required to add the plate to the object as a sub label.
  - Default: `0.9`.
- **`min_plate_length`**: Specifies the minimum number of characters a detected license plate must have to be added as a sub-label to an object.
  - Use this to filter out short, incomplete, or incorrect detections.
- **`format`**: A regular expression defining the expected format of detected plates. Plates that do not match this format will be discarded.
  - `"^[A-Z]{1,3} [A-Z]{1,2} [0-9]{1,4}$"` matches plates like "B AB 1234" or "M X 7"
  - `"^[A-Z]{2}[0-9]{2} [A-Z]{3}$"` matches plates like "AB12 XYZ" or "XY68 ABC"

### Matching

- **`known_plates`**: List of strings or regular expressions that assign custom a `sub_label` to `car` objects when a recognized plate matches a known value.
  - These labels appear in the UI, filters, and notifications.
- **`match_distance`**: Allows for minor variations (missing/incorrect characters) when matching a detected plate to a known plate.
  - For example, setting `match_distance: 1` allows a plate `ABCDE` to match `ABCBE` or `ABCD`.
  - This parameter will not operate on known plates that are defined as regular expressions. You should define the full string of your plate in `known_plates` in order to use `match_distance`.

### Examples

```yaml
lpr:
  enabled: True
  min_area: 1500 # Ignore plates smaller than 1500 pixels
  min_plate_length: 4 # Only recognize plates with 4 or more characters
  known_plates:
    Wife's Car:
      - "ABC-1234"
      - "ABC-I234" # Accounts for potential confusion between the number one (1) and capital letter I
    Johnny:
      - "J*N-*234" # Matches JHN-1234 and JMN-I234, but also note that "*" matches any number of characters
    Sally:
      - "[S5]LL-1234" # Matches both SLL-1234 and 5LL-1234
```

```yaml
lpr:
  enabled: True
  min_area: 4000 # Run recognition on larger plates only
  recognition_threshold: 0.85
  format: "^[A-Z]{3}-[0-9]{4}$" # Only recognize plates that are three letters, followed by a dash, followed by 4 numbers
  match_distance: 1 # Allow one character variation in plate matching
  known_plates:
    Delivery Van:
      - "RJK-5678"
      - "UPS-1234"
    Employee Parking:
      - "EMP-[0-9]{3}[A-Z]" # Matches plates like EMP-123A, EMP-456Z
```
