## n8n to Reve Configuration


#picture-with-picasso  #picasso 


can you provide specific json language to make it easier for you to handle both of my files with instructions in JSON format and any type of configuration details as well as provide information regarding the configuration of the outbound HTTP content that I need to receive back into my N8N workflow.

## Example outbound JSON body (n8n → Reve)

Assuming a “remix-edit” style endpoint that combines two images, your HTTP Request node in n8n can send JSON like this:​

{
  "model": "reve/remix-edit-image",
  "prompt": "Place the people from the first image standing next to the celebrity in the second image, matching lighting and perspective.",
  "image_urls": [
    "https://your-storage.example.com/images/people_foreground.jpg",
    "https://your-storage.example.com/images/celebrity_background.jpg"
  ],
  "aspect_ratio": "16:9",
  "num_images": 1,
  "output_format": "png",
  "convert_base64_to_url": true,
  "metadata": {
    "workflow_id": "n8n-reve-composite-01",
    "source_system": "n8n",
    "person_id": "12345",
    "celebrity_name": "Celebrity Name",
    "request_id": "{{ $json.requestId }}"
  },
  "webhook_url": "https://your-n8n-instance.example.com/webhook/reve-callback"
}


Key points for configuring the n8n HTTP Request node:​

- Method: POST.
    
- URL: the Reve remix/edit endpoint that accepts multiple image_urls (for example, one documented as accepting image_urls: []).
    
- Headers:
    
    - Authorization: Bearer {{ $env.REVE_API_KEY }} (or n8n Credentials).
        
    - Content-Type: application/json.
        
- Body: use “JSON” mode and map image_urls dynamically from previous nodes, or base64 fields if the API is defined that way.
    

If the endpoint instead expects a single image_url plus additional reference fields, you can switch image_urls to whatever the docs specify (for example, image_url and reference_image_url).​

## Example callback JSON body (Reve → n8n Webhook)

For an async pattern where Reve calls back to n8n via webhook_url, you can plan for a payload that looks like this:​

{
  "status": "succeeded",
  "request_id": "abcd-efgh-1234-5678",
  "model": "reve/remix-edit-image",
  "output": {
    "images": [
      {
        "url": "https://cdn.reve.com/generated/combined_people_celebrity_001.png",
        "content_type": "image/png",
        "width": 1920,
        "height": 1080,
        "file_size": 512345
      }
    ]
  },
  "metadata": {
    "workflow_id": "n8n-reve-composite-01",
    "source_system": "n8n",
    "person_id": "12345",
    "celebrity_name": "Celebrity Name",
    "request_id": "abcd-efgh-1234-5678"
  },
  "error": null
}

Webhook node configuration in n8n to receive this:​

- Method: POST.
    
- Response: 200 and a simple JSON or text confirmation.
    
- From the incoming JSON, you will:
    
    - Read status and error to branch success vs failure.
        
    - Extract output.images.url as the final composite image URL.
        
    - Use metadata.request_id or person_id to join back to any earlier data or to drive your video-generation step.
        

If you instead poll for results (no webhook_url), the same JSON shape can be expected as the response body from your “status” or “result” GET/POST request, and your HTTP Request node in n8n should parse that JSON similarly.[](https://fal.ai/models/fal-ai/reve/text-to-image/api)​

## Binary or base64 variant (if you do not use URLs)

If you want to send binary images directly from n8n and the API supports base64 data URIs, your JSON can look like this instead:​

Webhook node configuration in n8n to receive this:​

- Method: POST.
    
- Response: 200 and a simple JSON or text confirmation.
    
- From the incoming JSON, you will:
    
    - Read status and error to branch success vs failure.
        
    - Extract output.images.url as the final composite image URL.
        
    - Use metadata.request_id or person_id to join back to any earlier data or to drive your video-generation step.
        

If you instead poll for results (no webhook_url), the same JSON shape can be expected as the response body from your “status” or “result” GET/POST request, and your HTTP Request node in n8n should parse that JSON similarly.[](https://fal.ai/models/fal-ai/reve/text-to-image/api)​

## Binary or base64 variant (if you do not use URLs)

If you want to send binary images directly from n8n and the API supports base64 data URIs, your JSON can look like this instead:​

json

`{   "model": "reve/remix-edit-image",  "prompt": "Place the people from the first image standing next to the celebrity in the second image, matching lighting and perspective.",  "image_data": [    {      "name": "people_foreground",      "data": "data:image/jpeg;base64,{{ $json.foreground_base64 }}"    },    {      "name": "celebrity_background",      "data": "data:image/jpeg;base64,{{ $json.background_base64 }}"    }  ],  "num_images": 1,  "output_format": "png",  "convert_base64_to_url": true }`

In n8n you would add a Function or Set node to convert binary data to base64 strings and populate foreground_base64 and background_base64 before the HTTP Request.​

## What to configure for outbound and inbound HTTP in n8n

For outbound (to Reve):​

- HTTP Request node:
    
    - Method, URL, headers as above.
        
    - JSON body in one of the formats shown (image_urls or image_data).
        
    - Return type: JSON, so you can read request_id or immediate image URLs.
        

For inbound (from Reve back into n8n):​

- Webhook node (if async with callbacks):
    
    - Public URL pasted into webhook_url in the outbound JSON.
        
    - Expect JSON with fields like status, output.images, metadata.
        
- Or HTTP Request node (if you poll results):
    
    - URL: results endpoint with request_id.
        
    - Parse JSON as in the callback example, then pass output.images.url as binary or URL into your video generation HTTP node.
- 
