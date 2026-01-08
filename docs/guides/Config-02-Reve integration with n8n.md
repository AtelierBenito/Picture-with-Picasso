
how can integrate n8n with reve where i provide two images and have it combine it and then i pull it back into my n8n worklfow . How would i do this/

38 steps

I now have all the information needed to explain how to integrate n8n with Reve's Remix API. Let me provide you with a comprehensive answer.

# How to Integrate n8n with Reve to Combine Two Images

Based on the Reve API documentation, here's how you can set up an n8n workflow to send two images to Reve, have them combined, and pull the result back into your workflow.

## The API Endpoint You Need

Use the **Remix endpoint**: `POST https://api.reve.com/v1/image/remix`

This endpoint lets you provide up to 6 reference images and use `<img>0</img>`, `<img>1</img>` tags in your prompt to reference and combine them.

## n8n Workflow Setup

### Step 1: Get a Reve API Key

Go to [https://api.reve.com/console](https://api.reve.com/console) and click "Start building" to get your API key.

### Step 2: Create Your n8n Workflow

**Node 1: Trigger** (however you want to start the workflow - manual, webhook, schedule, etc.)

**Node 2: Convert Images to Base64** Use a **Code node** or **Function node** to convert your two images to base64 format. If your images come from URLs, you can use an **HTTP Request node** first to fetch them.

**Node 3: HTTP Request to Reve API** Configure an HTTP Request node with:

- **Method**: POST
- **URL**: `https://api.reve.com/v1/image/remix`
- **Headers**:
    - `Authorization`: `Bearer YOUR_REVE_API_KEY`
    - `Content-Type`: `application/json`
    - `Accept`: `application/json`
- **Body (JSON)**:

json

```json
{
  "prompt": "Combine the subject from <img>0</img> with the background from <img>1</img>",
  "reference_images": [
    "base64_encoded_image_1_here",
    "base64_encoded_image_2_here"
  ],
  "aspect_ratio": "1:1",
  "version": "latest"
}
```

### Step 3: Handle the Response

The API returns a JSON object with:

- `image`: The base64-encoded combined image
- `version`: Model version used
- `credits_used`: Credits consumed
- `credits_remaining`: Your remaining balance

### Example Prompt Patterns

- **Style transfer**: `"Apply the style from <img>0</img> to the content in <img>1</img>"`
- **Subject replacement**: `"Place the person from <img>0</img> in the scene from <img>1</img>"`
- **Blending**: `"Blend the elements of <img>0</img> with <img>1</img> creating a seamless composition"`

## Key Requirements

- **Images must be base64 encoded** (without the data URL prefix)
- Each image must be **less than 10 MB**
- Total pixel count across all images: **max 32 million pixels**
- You can use **1-6 reference images**
- **Aspect ratios**: 16:9, 9:16, 3:2, 2:3, 4:3, 3:4, or 1:1

## Sample n8n Code Node for Base64 Conversion

javascript

```javascript
// If images come as binary data or URLs
const image1Base64 = $input.first().binary.image1.data;
const image2Base64 = $input.first().binary.image2.data;

return {
  json: {
    prompt: "Combine the subject from <img>0</img> with the style of <img>1</img>",
    reference_images: [image1Base64, image2Base64],
    aspect_ratio: "1:1",
    version: "latest"
  }
};
```