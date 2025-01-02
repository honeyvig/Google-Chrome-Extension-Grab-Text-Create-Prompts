# Google-Chrome-Extension-Grab-Text-Create-Prompts
This is a Google Chrome extension that allows users to grab any text from a website and use AI to describe the text in written form. This will be helpful for creating AI designs later on.

Users should be able to prepare a text prompt by right-clicking on a webpage, identifying the image they want to describe, and having the application generate a text prompt. This prompt can then be used in programs like MidJourney and Ideogram for text-to-image AI generation.
-------
To create a Google Chrome extension that grabs text from a webpage and uses AI to describe it for generating text prompts for text-to-image AI tools (like MidJourney and Ideogram), we need to implement the following functionality:
Features:

    Right-click Context Menu: Users can right-click on a webpage to capture text from a specific area.
    Use AI for Description: The extension will send the captured text to an AI service (e.g., OpenAI's GPT-3/4) to generate a description of that text.
    Generate Prompt: The description will then be formatted into a usable prompt for text-to-image models.
    UI for displaying generated prompts: A simple popup will show the generated text that can be copied or exported.

Steps to implement:

    Create the manifest file (which defines the extension’s structure and permissions).
    Develop the background script to handle user actions (like right-click).
    Implement the content script to extract selected text from the webpage.
    Call the AI API (e.g., OpenAI GPT) to generate a descriptive text prompt.
    Show the generated prompt in the extension’s popup UI.
    Allow users to copy the generated prompt.

Here’s the code to implement this extension:
1. manifest.json - The Extension Manifest

{
  "manifest_version": 3,
  "name": "AI Text-to-Image Generator",
  "description": "Grab text from any webpage and generate an AI description for text-to-image models.",
  "version": "1.0",
  "permissions": [
    "activeTab",
    "contextMenus",
    "storage"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "images/icon16.png",
      "48": "images/icon48.png",
      "128": "images/icon128.png"
    }
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"]
    }
  ],
  "icons": {
    "16": "images/icon16.png",
    "48": "images/icon48.png",
    "128": "images/icon128.png"
  }
}

2. background.js - Handle Right-click Context Menu and Call AI API

chrome.runtime.onInstalled.addListener(() => {
  chrome.contextMenus.create({
    id: "generatePrompt",
    title: "Generate AI Description",
    contexts: ["selection"]  // Only show for selected text
  });
});

// When the context menu item is clicked, grab selected text and process it
chrome.contextMenus.onClicked.addListener(async (info) => {
  const selectedText = info.selectionText;
  if (selectedText) {
    const aiResponse = await generateDescription(selectedText);
    chrome.storage.local.set({ aiDescription: aiResponse });
  }
});

// Function to call OpenAI API to generate description (use GPT-3/4)
async function generateDescription(text) {
  const apiKey = "YOUR_OPENAI_API_KEY";  // Replace with your OpenAI API Key
  const response = await fetch("https://api.openai.com/v1/completions", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${apiKey}`
    },
    body: JSON.stringify({
      model: "text-davinci-003",  // You can replace with a newer model
      prompt: `Generate a detailed text prompt for text-to-image AI based on the following content: ${text}`,
      max_tokens: 150
    })
  });
  const data = await response.json();
  return data.choices[0].text.trim();
}

3. content.js - Extract Text from the Webpage

// This script will extract the selected text from the page.
document.addEventListener("mouseup", function () {
  const selectedText = window.getSelection().toString();
  if (selectedText.length > 0) {
    chrome.storage.local.set({ selectedText });
  }
});

4. popup.html - Display the Generated Prompt

<!DOCTYPE html>
<html>
  <head>
    <title>AI Description</title>
    <style>
      body {
        font-family: Arial, sans-serif;
        padding: 10px;
        width: 300px;
      }
      button {
        margin-top: 10px;
        padding: 8px;
        background-color: #4CAF50;
        color: white;
        border: none;
        cursor: pointer;
      }
      button:hover {
        background-color: #45a049;
      }
      #prompt {
        margin-top: 10px;
        padding: 8px;
        width: 100%;
        height: 100px;
        font-size: 14px;
        resize: none;
      }
    </style>
  </head>
  <body>
    <h3>Generated Prompt</h3>
    <textarea id="prompt" readonly></textarea>
    <button id="copyButton">Copy Prompt</button>

    <script>
      // Get AI description from storage and display it in the popup
      chrome.storage.local.get("aiDescription", function (data) {
        if (data.aiDescription) {
          document.getElementById("prompt").value = data.aiDescription;
        } else {
          document.getElementById("prompt").value = "No description generated yet.";
        }
      });

      // Copy prompt text to clipboard
      document.getElementById("copyButton").addEventListener("click", function () {
        const promptText = document.getElementById("prompt");
        promptText.select();
        document.execCommand("copy");
      });
    </script>
  </body>
</html>

5. Images Folder

You'll need to include images for the extension icon in the images/ folder. These images should be named as icon16.png, icon48.png, and icon128.png.
Instructions for Setup:

    Install the Extension:
        Save the above files in a folder, e.g., ai-text-prompt-extension.
        Open Chrome and go to chrome://extensions.
        Enable Developer Mode (top right).
        Click on "Load unpacked" and select the folder where you saved the files.
        The extension will now be installed.

    Using the Extension:
        Navigate to any webpage with text.
        Select the text you want to describe.
        Right-click on the selected text and choose "Generate AI Description".
        A text prompt will be generated and saved.
        Click on the extension's icon and view the generated prompt in the popup.
        You can copy the prompt to use it for text-to-image AI tools like MidJourney and Ideogram.

    API Key Configuration:
        Replace YOUR_OPENAI_API_KEY with your actual OpenAI API key in background.js.

Conclusion

This Chrome extension captures text from a webpage, sends it to an AI model (via OpenAI), and generates a descriptive text prompt. It integrates seamlessly with your browser to assist with generating prompts for text-to-image AI models like MidJourney or Ideogram. You can enhance the functionality by adding more features or fine-tuning the prompts based on the type of content on the page.
