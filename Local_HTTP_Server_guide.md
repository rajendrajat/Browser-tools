## Option B – Local HTTP Server Setup for .eml Download Testing

### Objective
Simulate downloading a `.eml` file over HTTP in Microsoft Edge to trigger SmartScreen / download warning.

---

### Step 1: Create Test File

1. Open Notepad  
2. Add sample content:
   ```
   From: test@test.com
   To: user@test.com
   Subject: Test Mail

   This is a test email file
   ```

3. Save the file as:
   ```
   test.eml
   ```

---

### Step 2: Prepare Hosting Folder

1. Create a folder (example):
   ```
   C:\Edge-Test\lab
   ```

2. Place `test.eml` inside this folder

---

### Step 3: Start Local HTTP Server

1. Open Command Prompt  
2. Navigate to the folder:
   ```
   cd C:\Edge-Test\lab
   ```

3. Start the server:
   ```
   python -m http.server 8000
   ```

This will serve all files in this folder over HTTP

---

### Step 4: Access the File from Edge

Open Microsoft Edge and go to:

- Same machine:
  ```
  http://localhost:8000/test.eml
  ```

- From another machine (same network):
  ```
  http://<your-IP>:8000/test.eml
  ```

---

### Step 5: Perform Download Test

1. Click on the file link  
2. Allow it to download  
3. Observe download behavior

---

### Expected Result

Edge may show warning such as:
- "This file isn’t commonly downloaded"
- "This file may be unsafe"

Options:
- Keep
- Discard

---

### Important Notes

- Server serves files from the same folder only
- Always download using `http://` (not file://)
- Ensure Microsoft Defender SmartScreen is enabled in Edge

---

### Troubleshooting (if warning not triggered)

- Rename file (example: `invoice_urgent.eml`)
- Download from another machine
- Zip the file and test (`.zip → .eml`)
- Try hosting on OneDrive / external source

---

### Key Insight

`.eml` files may trigger warnings because:
- Low download reputation  
- Less commonly used format  
- Potential embedded content  

This is expected behavior in most cases, not a product issue
