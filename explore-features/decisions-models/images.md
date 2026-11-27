---
description: >-
  Send images to decision models on either endpoint: as evidence to evaluate,
  and, with TypeSafe Jev, inside a question's instructions or as the criteria it
  is judged against.
icon: gavel
---

# Images

> **Jev image support is in preview.** TypeSafe describes image input on Jev as ready for prototypes rather than production. The request format, answers, and limits on this page can change between Jev releases, and accuracy will change as the models are updated.&#x20;

### Model support

| Model                 | Images on /decisions | Images on /systemone                          |
| --------------------- | -------------------- | --------------------------------------------- |
| `typesafe/jev-latest` | In `input`           | Anywhere: `state`, `instructions`, `criteria` |
| `typesafe/jev-1.14.0` | In `input`           | Anywhere: `state`, `instructions`, `criteria` |
| `typesafe/jev-1.13.0` | In `input`           | Anywhere: `state`, `instructions`, `criteria` |

Each endpoint has its own way of attaching images:

| Endpoint     | How to send an image                                                                        |
| ------------ | ------------------------------------------------------------------------------------------- |
| `/decisions` | An `input_image` part with an inline base64 **data URL**, inside a user message in `input`. |
| `/systemone` | An entry in the top-level `assets` map, referenced by a string wherever the image belongs.  |

***

### Images on `/decisions`

Put the image in a user message in `input`, as an `input_image` part whose `image_url` is a `data:` URL. Combine it with `input_text` parts to give context. Every question in the request is evaluated against it.

{% tabs %}
{% tab title="cURL" %}
```bash
IMAGE_BASE64="$(base64 < product.jpg | tr -d '\r\n')"

curl https://api.fastrouter.ai/api/v1/decisions \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -H "TypeSafe-Danger-Zone-Images: true" \
  --data-binary @- <<JSON
{
  "model": "typesafe/jev-latest",
  "input": [{
    "role": "user",
    "content": [
      {"type": "input_text", "text": "Inspect the product in this photo."},
      {"type": "input_image", "image_url": "data:image/jpeg;base64,$IMAGE_BASE64"}
    ]
  }],
  "questions": [
    {
      "type": "predicate",
      "name": "visible_damage",
      "instructions": "Does the product have visible damage, such as a crack, tear, or dent? Ignore shadows and damage to the packaging."
    },
    {
      "type": "choice",
      "name": "category",
      "instructions": "What kind of product is shown?",
      "choices": [
        {"value": "kitchenware", "description": "Mugs, plates, cookware."},
        {"value": "electronics", "description": "Devices, cables, chargers."},
        {"value": "other", "description": "Anything else."}
      ]
    }
  ]
}
JSON
```
{% endtab %}

{% tab title="Python" %}
```python
import base64
import os

import requests

with open("product.jpg", "rb") as f:
    image_b64 = base64.b64encode(f.read()).decode("ascii")

resp = requests.post(
    "https://api.fastrouter.ai/api/v1/decisions",
    headers={
        "Authorization": f"Bearer {os.environ['FASTROUTER_API_KEY']}",
        "TypeSafe-Danger-Zone-Images": "true",  # needed for Jev; ignored by gpt-6-luna
    },
    json={
        "model": "typesafe/jev-latest",  # or "gpt-6-luna"
        "input": [
            {
                "role": "user",
                "content": [
                    {"type": "input_text", "text": "Inspect the product in this photo."},
                    {"type": "input_image", "image_url": f"data:image/jpeg;base64,{image_b64}"},
                ],
            }
        ],
        "questions": [
            {
                "type": "predicate",
                "name": "visible_damage",
                "instructions": "Does the product have visible damage, such as a crack, tear, or dent?",
            }
        ],
    },
    timeout=60,
)
resp.raise_for_status()
print(resp.json()["answers"][0]["probability"])
```
{% endtab %}
{% endtabs %}

***

### Images on `/systemone`

Each image is listed once, in the request's top-level `assets` map. Its key in that map is a **reference string**. Write the reference wherever the image belongs, as a whole string value or inside a longer string, and the model receives the image in its place.

This request sends a photo of a found dog as the state, and puts the photo from a lost-dog poster directly into one question:

```json
{
  "model": "typesafe/jev-latest",
  "state": {
    "photo": "a3f1c2e8-found"
  },
  "assets": {
    "a3f1c2e8-found": {"type": "image", "format": "jpg", "data_base64": "/9j/4AAQSkZJRg..."},
    "77b0d4a1-poster": {"type": "image", "format": "jpg", "data_base64": "/9j/4AAQSkZJRg..."}
  },
  "questions": {
    "same_dog": {
      "type": "noul",
      "instructions": "Is the dog in `photo` the same dog as in this picture: 77b0d4a1-poster?"
    },
    "has_collar": {
      "type": "noul",
      "instructions": "Is the dog in `photo` wearing a collar?"
    },
    "coat": {
      "type": "choice",
      "instructions": "What is the coat of the dog in `photo` like?",
      "criteria": {
        "short": "Short and smooth.",
        "long": "Long or feathered.",
        "curly": "Curly or wiry."
      }
    }
  }
}
```

Questions can refer to an image in the state by name or path in backticks, the same way they refer to any other field. An image that only one question uses can go directly in that question's `instructions` or `criteria`, as the poster photo does above (Jev models only).

#### Full example

{% tabs %}
{% tab title="cURL" %}
```bash
DOOR_B64="$(base64 < door-camera.jpg | tr -d '\r\n')"

curl https://api.fastrouter.ai/api/v1/systemone \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -H "TypeSafe-Danger-Zone-Images: true" \
  --data-binary @- <<JSON
{
  "model": "typesafe/jev-latest",
  "state": {"door": "a3f1c2e8-door"},
  "assets": {
    "a3f1c2e8-door": {"type": "image", "format": "jpg", "data_base64": "$DOOR_B64"}
  },
  "questions": {
    "person_present": {"type": "noul", "instructions": "Is there a person in \`door\`?"}
  }
}
JSON
```
{% endtab %}

{% tab title="Python" %}
```python
import base64
import os
import uuid

import requests


def image_asset(path: str) -> tuple[str, dict]:
    """Return a new reference and the asset entry for one image file."""
    fmt = "png" if path.lower().endswith(".png") else "jpg"
    with open(path, "rb") as f:
        data = base64.b64encode(f.read()).decode("ascii")
    return str(uuid.uuid4()), {"type": "image", "format": fmt, "data_base64": data}


photo_key, photo = image_asset("door-camera.jpg")

response = requests.post(
    "https://api.fastrouter.ai/api/v1/systemone",
    headers={
        "Authorization": f"Bearer {os.environ['FASTROUTER_API_KEY']}",
        "TypeSafe-Danger-Zone-Images": "true",
    },
    json={
        "model": "typesafe/jev-latest",
        "state": {"door": photo_key},
        "assets": {photo_key: photo},
        "questions": {
            "person_present": {
                "type": "noul",
                "instructions": "Is there a person in `door`?",
            },
            "carrying": {
                "type": "choice",
                "instructions": "What is the person in `door` carrying?",
                "criteria": {
                    "package": "A box or parcel.",
                    "bag": "A bag of groceries or shopping.",
                    "nothing": "Empty hands.",
                },
            },
        },
    },
    timeout=60,
)
response.raise_for_status()
print(response.json()["answers"])
```
{% endtab %}
{% endtabs %}

#### Asset fields

**`assets`** `map<string, Asset>`

The images in the request, keyed by reference string. Every reference must appear somewhere in the `state`, `instructions`, or `criteria`.

| Field         | Type                           | Required | Description                                                                                                                  |
| ------------- | ------------------------------ | -------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `type`        | `"image"`                      | No       | The kind of asset. If you send it, it must be `"image"`.                                                                     |
| `data_base64` | string                         | Yes      | The image file, base64-encoded. Standard alphabet with `=` padding, no line breaks or whitespace, and **no `data:` prefix**. |
| `format`      | `"png"` \| `"jpg"` \| `"jpeg"` | Yes      | The image's format, in lowercase. `"jpeg"` is another name for `"jpg"`.                                                      |

> Note the difference between endpoints: `/decisions` takes a full `data:image/...;base64,` URL, while `/systemone` takes the bare base64 string in `data_base64` plus a separate `format`.&#x20;

#### References

References are matched as plain text anywhere in the request's strings, so a reference works mid-sentence as well as as a whole value.

* **Use references that can't occur by accident.** A short reference such as `cat` could also match inside `category`. A new UUID for each image in each request is the safest choice.
* **Every asset must be referenced.** A request with an asset that is not referenced anywhere returns `400`.
* **Whole values match first.** A string that is exactly a reference becomes that image. In longer strings, references are replaced from left to right, and matches never overlap.
* **Each use adds the image again.** The file is uploaded once, but every place its reference appears adds another copy of the image to what the model reads, and every copy counts toward input tokens.
* **Object keys are not replaced.** A choice option name, a question key, or any other object key stays text. To define a choice option with a picture, put the reference in the option's description, as in [Criteria](images.md#criteria-jev-only).
* **Answers contain the reference, not the image,** wherever they repeat your criteria, such as a score's `legend`.
* **Keep references short.** Each reference can be at most 1,024 bytes of UTF-8, and all references in a request together at most 65,536 bytes. A reference can't be empty or only whitespace.

***

### Image placement on `/systemone`

#### State

Images in the `state` are part of the state, the same as any text or JSON there. Every question in the request is evaluated against them. A state can mix images with text, and can hold several images, labeled or nested like any other state data. This state holds a short history of camera frames, oldest first:

```json
{
  "state": {
    "frames": [
      {"seconds_ago": 2, "frame": "5c0e9a12-f1"},
      {"seconds_ago": 1, "frame": "1d7be0c4-f2"},
      {"seconds_ago": 0, "frame": "e48a6f30-f3"}
    ]
  }
}
```

A question can ask "Did the door in `frames` open during these frames?" or refer to one frame by path, such as `` `frames[2].frame` ``.

#### Instructions (Jev only)

An image in a question's `instructions` is part of that question and is sent only with that question. Put the reference in the sentence where the question needs the image:

```json
{
  "model": "typesafe/jev-latest",
  "state": {"door": "e48a6f30-door"},
  "questions": {
    "is_badge_holder": {
      "type": "noul",
      "instructions": "Is the person in `door` the same person as in this badge photo: 9a21c7d3-badge?"
    }
  }
}
```

An image in `instructions` is sent to the model once for every question that includes it. When several questions use the same image, put it in the `state` once and refer to it by path from each question. That uses fewer tokens and less of the size limits.

#### Criteria (Jev only)

Choice options, score levels, and noul `true` and `false` definitions can each include images. This choice classifies a hand gesture, with each option defined by example pictures. The option names are the values your code receives in `choice`, and the pictures are what the model compares the hand against:

```json
{
  "model": "typesafe/jev-latest",
  "state": {"camera": "e48a6f30-cam"},
  "questions": {
    "gesture": {
      "type": "choice",
      "instructions": "Which gesture is the hand in `camera` making?",
      "criteria": {
        "thumbs_up": "0b6c51aa-thumb",
        "open_palm": {
          "description": "A hand held up with the palm facing the camera.",
          "examples": ["4fd2e7b9-palm1", "c83a0e15-palm2"]
        },
        "fist": "a7e94d02-fist",
        "none": "No hand, or a hand that matches none of the examples."
      }
    }
  }
}
```

An option can be an image on its own (`thumbs_up`), a description with one or more examples (`open_palm`), or text only (`none`). The `description` and `examples` keys are names chosen for this example, not required fields; criteria accept any structure.

Score and noul criteria take images the same way:

* **Score levels** can each include an example picture. A score for how open a hand is could use a closed fist, a half-open hand, and an open hand as its three levels.
* **Noul criteria** can define `true` and `false` with pictures: `{"true": {"description": "Correctly assembled", "examples": ["<reference>"]}, "false": "Parts missing or in the wrong place."}`.

#### Structured data

State, instructions, and criteria accept nested JSON, and a reference works at any level of it. A question can refer to one picture in an array as `` `lineup[2]` ``, or to one in a labeled object as `` `faces.happy` ``. A criterion can hold several example pictures as an array.

***

### Patterns

| Pattern                         | How it works                                                                                                                          | Where the image goes                              |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| Compare against a known picture | Ask whether the state shows the same person, product, or place as a known picture.                                                    | Known picture in `instructions`, photo in `state` |
| Define options by example       | Define each choice option with one or a few example pictures.                                                                         | `criteria`                                        |
| What changed                    | Send the previous frame or a short frame history, and ask about motion, arrivals, or differences.                                     | `state`                                           |
| Look closer when needed         | Ask about a small copy of the image first. Send a larger copy, or a full-resolution crop, only when the answer needs more detail.     | `state`                                           |
| Search a large image            | Cut the image into tiles and ask a choice with the tiles as its options. Repeat on the chosen tile until the target fills most of it. | `criteria`                                        |

The patterns that use `instructions` or `criteria` need a Jev model on `/systemone`.

***

### Limits

These limits apply to images sent to Jev models.

<table data-search="false"><thead><tr><th>Limit</th><th>Value</th></tr></thead><tbody><tr><td>Format</td><td>PNG or JPEG, still images only. Animated PNGs are rejected.</td></tr><tr><td>File size</td><td>16 MiB per image file. For base64, this is the size after decoding.</td></tr><tr><td>Width and height</td><td>280 to 1568 pixels each. An image with either side under 280 pixels is rejected.</td></tr><tr><td>Total pixels (width × height)</td><td>78,400 to 2,458,624 (the area of a 280 × 280 to a 1568 × 1568 square).</td></tr><tr><td>Request body</td><td>32 MiB, including the JSON and base64 text. HTTP headers don't count.</td></tr><tr><td>All images in the request</td><td>32 MiB, shared by every distinct image after processing. Processing re-encodes each image and can make it larger than the file you sent.</td></tr><tr><td>Expanded request</td><td>32 MiB for the text, questions, structure, and images after every reference is replaced by its image. An image that is 10 MiB after processing, referenced four times, is over the limit.</td></tr></tbody></table>

All limits include both ends of the range. An image outside them is rejected, never resized to fit. If any image in a request is rejected, the whole request fails: there are no partial answers.

Base64 makes each image about a third larger, and that extra size counts toward the 32 MiB request body limit. To stay under the limits, send fewer or smaller images, put an image that several questions use in the `state` once instead of in each question's `instructions`, or split the work across requests.

***

### Preparing images

Send each image as a PNG or JPEG, rotated upright and sized within the [limits](images.md#limits):

* **Resize before sending.** Scale the image so its longest side is at most 1568 pixels. A typical 12-megapixel phone photo becomes 1568 × 1176. Scale up, or pad, an image smaller than the minimum.
* **Rotate the pixels upright.** Metadata (EXIF, color profiles, comments) is removed and the EXIF orientation is not applied. A photo that looks upright in your photo viewer can reach the model sideways, so rotate the pixels themselves first.
* **Send a still image.** For an animation or a video, export the frame you want. Convert GIF, WebP, HEIC, and other formats to PNG or JPEG.
* **Prefer standard JPEG.** A standard 8-bit baseline JPEG is the safest JPEG to send. Re-save progressive JPEGs as baseline.
* **Re-export a rejected file.** If a file that opens fine on your computer is rejected, open it in a standard image editor or library and export it again as PNG or JPEG.

This Python example uses Pillow to prepare one photo:

```python
from PIL import Image, ImageOps

with Image.open("photo.jpg") as original:
    img = ImageOps.exif_transpose(original)  # rotate the pixels upright
    img.thumbnail((1568, 1568))  # longest side at most 1568, same aspect ratio
    # thumbnail only shrinks: upscale or pad an image under the minimum separately
    img.convert("RGB").save("photo-ready.jpg", "JPEG", quality=90)  # baseline JPEG
```

***

### Image tokens and billing

Each image counts as input tokens, billed at the model's normal per-token input price. There is no separate charge for images. `usage.input_tokens` in the response includes the tokens for every image in the request, and image tokens count toward the model's context length.

***

### Errors

| Status | Cause                                                                                                                                                                                                          |
| ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `400`  | Images are not turned on: a request sends an image to a Jev model without the `TypeSafe-Danger-Zone-Images: true` header.                                                                                      |
| `400`  | The model does not accept images in that position, such as an image sent to `gemini-decision-1`, or an image in `instructions` or `criteria` sent to `gpt-6-luna`.                                             |
| `400`  | An image or asset was rejected: the file isn't PNG or JPEG, is damaged or animated, is below the minimum size; an asset isn't referenced anywhere; the base64 or `format` is invalid; or a reference is empty. |
| `400`  | `max_tokens_exceeded`: the images and text together exceed the model's context length. Send fewer or smaller images, or move images out of `instructions` into the `state`.                                    |
| `413`  | The request is over a size limit: an image file, an image's dimensions, the request body, all images together, the expanded request, or the reference lengths. See [Limits](images.md#limits).                 |

If a request has several problems, the error reports one of them. Check the request against [Limits](images.md#limits) and [Preparing images](images.md#preparing-images).
