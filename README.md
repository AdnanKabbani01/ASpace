# ASpace


Hosting your Spring Boot application on GitHub can be done using GitHub Pages for static content or using other platforms for hosting the backend Spring Boot application. Here’s a general approach to host your Spring Boot application on GitHub:

### 1. Prepare Your Spring Boot Application

Before hosting your application, ensure it's ready for deployment:

- Make sure your Spring Boot application (`ASpaceApplication`) is fully functional and tested locally.
- Ensure all necessary configurations (like Google Cloud Storage integration) are properly set up.
- Test the application to verify it works as expected, including file uploads, downloads, and WebSocket communication.

### 2. Set Up GitHub Repository

If you haven't already, create a GitHub repository to host your Spring Boot application:

- Go to GitHub and create a new repository for your project.
- Clone the repository to your local machine if you haven't already done so:
  ```bash
  git clone https://github.com/yourusername/your-repository.git
  ```
- Move your Spring Boot application files into this repository.

### 3. Configure GitHub Pages (Optional for Frontend)

If your Spring Boot application includes static content (like HTML, CSS, and JavaScript files for frontend), you can use GitHub Pages to host these static files:

- Create a `docs` folder in your repository root (or use an existing folder).
- Place your static files (HTML, CSS, JavaScript) in this `docs` folder.
- Enable GitHub Pages from your repository settings:
  - Go to your repository on GitHub.
  - Navigate to "Settings" -> "Pages".
  - Choose the branch (`main` or `master`) and folder (`/docs`) from which GitHub Pages will be served.
  - Save the settings to enable GitHub Pages.

### 4. Deploy Backend (Spring Boot Application)

Since GitHub Pages is for static content, you'll need an alternative approach to deploy your Spring Boot backend:

- **Heroku**: Deploy your Spring Boot application to Heroku, which offers free hosting options for Java applications.
- **AWS Elastic Beanstalk**: Deploy using AWS Elastic Beanstalk, which provides easy deployment and scaling for Java applications.
- **DigitalOcean**: Deploy using DigitalOcean Droplets or Kubernetes for more control and flexibility.

Choose a platform that suits your needs and integrates well with your Spring Boot application.

### 5. Continuous Deployment (Optional)

Set up continuous deployment to automatically deploy changes to your GitHub-hosted repository:

- Use GitHub Actions or other CI/CD tools to automate build and deployment processes.
- Configure workflows to build and deploy your Spring Boot application whenever changes are pushed to the repository.

### 6. Update Documentation and README

Update your repository’s README file with instructions on how to use and access your application once deployed. Include information on how to access endpoints, URLs, and any necessary setup instructions.

### Example Workflow

Here’s a simplified example workflow assuming you're using Heroku for backend deployment and GitHub Pages for frontend:

- **Frontend (HTML/CSS/JS)**:
  - Place static files (HTML, CSS, JS) in



To fully integrate Google Cloud Storage (GCS) into your Spring Boot application for file storage and retrieval, follow these steps:

### 1. Set Up Google Cloud Storage

Before integrating GCS into your Spring Boot application, you need to set up a Google Cloud Platform (GCP) project and enable Google Cloud Storage:

- **Create a GCP Project**: Go to the [Google Cloud Console](https://console.cloud.google.com/) and create a new project if you haven't already.

- **Enable Google Cloud Storage API**: Navigate to the [Google Cloud Storage](https://console.cloud.google.com/storage) section in the Cloud Console and enable the Google Cloud Storage API for your project.

- **Create a Bucket**: Create a new bucket in Google Cloud Storage where your application will store files. Note down the bucket name and ensure proper permissions are set (if accessing publicly or through authenticated users).

### 2. Add Dependencies

Add the necessary dependencies to your `pom.xml` for integrating with Google Cloud Storage:

```xml
<dependency>
    <groupId>com.google.cloud</groupId>
    <artifactId>google-cloud-storage</artifactId>
    <version>2.4.0</version> <!-- Replace with the latest version -->
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-gcp-starter-storage</artifactId>
</dependency>
```

### 3. Configure GCS Credentials

Configure credentials to authenticate your Spring Boot application with Google Cloud Storage:

- **Service Account Key**: Create a service account in your GCP project and download the JSON key file.
- **Environment Variable**: Set an environment variable to point to the location of your service account key file:

  ```bash
  export GOOGLE_APPLICATION_CREDENTIALS="/path/to/your/service-account-file.json"
  ```

  Replace `/path/to/your/service-account-file.json` with the actual path to your JSON key file.

### 4. Implement File Upload and Download

Create services or controllers to handle file upload and download operations using Google Cloud Storage.

#### Service for Google Cloud Storage Operations

Create a service (`StorageService`) to encapsulate interactions with Google Cloud Storage:

```java
import com.google.cloud.storage.Blob;
import com.google.cloud.storage.Storage;
import com.google.cloud.storage.StorageOptions;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import java.io.IOException;

@Service
public class StorageService {

    @Value("${google.cloud.project-id}")
    private String projectId;

    @Value("${google.cloud.storage.bucket-name}")
    private String bucketName;

    private Storage storage;

    public StorageService() {
        storage = StorageOptions.newBuilder().setProjectId(projectId).build().getService();
    }

    public void uploadFile(MultipartFile file) throws IOException {
        Blob blob = storage.create(
                BlobInfo.newBuilder(bucketName, file.getOriginalFilename()).build(),
                file.getInputStream());
    }

    public Blob getFile(String fileName) {
        return storage.get(bucketName, fileName);
    }
}
```

#### Controller for File Upload and Download

Create a controller (`StorageController`) to handle HTTP requests for file upload and download:

```java
import com.google.cloud.storage.Blob;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import java.io.IOException;

@RestController
@RequestMapping("/api/storage")
public class StorageController {

    @Autowired
    private StorageService storageService;

    @PostMapping("/upload")
    public ResponseEntity<String> handleFileUpload(@RequestParam("file") MultipartFile file) {
        try {
            storageService.uploadFile(file);
            return ResponseEntity.ok().body("File uploaded successfully: " + file.getOriginalFilename());
        } catch (IOException e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("Failed to upload file.");
        }
    }

    @GetMapping("/download/{fileName}")
    public ResponseEntity<byte[]> downloadFile(@PathVariable String fileName) {
        Blob blob = storageService.getFile(fileName);
        if (blob != null) {
            byte[] content = blob.getContent();
            // Add headers for file download
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
            headers.setContentDispositionFormData("attachment", fileName);
            return ResponseEntity.ok().headers(headers).body(content);
        } else {
            return ResponseEntity.notFound().build();
        }
    }
}
```

### 5. Configure Application Properties

Configure your application properties (`application.properties` or `application.yml`) with Google Cloud Storage settings:

```properties
# Google Cloud Storage configuration
google.cloud.project-id=your-project-id
google.cloud.storage.bucket-name=your-bucket-name
```

Replace `your-project-id` and `your-bucket-name` with your actual GCP project ID and Google Cloud Storage bucket name.

### 6. Test and Deploy

- Test your application locally to ensure file upload and download operations work with Google Cloud Storage.
- Once tested, deploy your Spring Boot application to your preferred platform (Heroku, AWS, etc.) and ensure it's accessible.

By following these steps, you can fully integrate Google Cloud Storage into your Spring Boot application for secure and scalable file storage operations. Adjust the configurations and code based on your specific requirements and security considerations.


To create a simple HTML frontend for your file upload and download application using Spring Boot and Google Cloud Storage, you can follow these steps. This frontend will allow users to upload files and view/download available files.

### HTML Frontend

Create an HTML file (`index.html`) in your `src/main/resources/static` directory. This example assumes you are using Thymeleaf for template rendering, as per your previous code snippets.

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>File Upload and Download</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin-top: 50px;
        }
        .container {
            display: inline-block;
            text-align: left;
        }
        .file-list {
            list-style-type: none;
            padding: 0;
        }
        .file-list li {
            margin-bottom: 10px;
        }
        .file-list li a {
            text-decoration: none;
            color: #007bff;
        }
        .file-list li a:hover {
            text-decoration: underline;
        }
    </style>
</head>
<body>
<div class="container">
    <h1>File Upload and Download</h1>
    
    <!-- File Upload Form -->
    <form id="fileUploadForm" enctype="multipart/form-data">
        <input type="file" id="fileInput" name="file"/>
        <button type="button" onclick="uploadFile()">Upload File</button>
    </form>
    
    <h2>Available Files</h2>
    <ul id="fileList" class="file-list">
        <!-- Files will be dynamically added here -->
    </ul>
</div>

<script th:inline="javascript">
    // WebSocket connection to receive notifications about new files
    const sessionId = /*[[${sessionId}]]*/ 'your-session-id';
    const socket = new WebSocket('ws://localhost:8080/ws/' + sessionId);

    socket.onmessage = function(event) {
        const message = event.data;
        if (message.startsWith('File available: ')) {
            const fileName = message.substring('File available: '.length);
            addFileToList(fileName);
        }
    };

    // Function to upload file using AJAX
    function uploadFile() {
        const fileInput = document.getElementById('fileInput');
        const file = fileInput.files[0];
        const formData = new FormData();
        formData.append('file', file);

        fetch('/api/storage/upload', {
            method: 'POST',
            body: formData
        })
        .then(response => response.text())
        .then(data => {
            console.log(data);
            alert('File uploaded successfully');
        })
        .catch(error => {
            console.error('Error:', error);
            alert('File upload failed');
        });
    }

    // Function to add file to the list after upload
    function addFileToList(fileName) {
        const fileList = document.getElementById('fileList');
        const fileItem = document.createElement('li');
        const fileLink = document.createElement('a');
        fileLink.href = '/api/storage/download/' + fileName;
        fileLink.textContent = fileName;
        fileItem.appendChild(fileLink);
        fileList.appendChild(fileItem);
    }

    // Fetch initial list of available files
    fetch('/api/storage/files')
        .then(response => response.json())
        .then(files => {
            for (const fileName of files) {
                addFileToList(fileName);
            }
        });
</script>

</body>
</html>
```

### Explanation:

1. **File Upload Form**: 
   - Creates a form (`fileUploadForm`) with a file input field (`fileInput`) and a button (`Upload File`). Clicking the button triggers the `uploadFile()` JavaScript function.

2. **Available Files List**: 
   - Uses a `<ul>` element (`fileList`) to dynamically list available files. Initially populated by fetching data from `/api/storage/files` endpoint.

3. **WebSocket Connection**: 
   - Connects to a WebSocket endpoint (`/ws/{sessionId}`) to receive notifications about new files uploaded.

4. **JavaScript Functions**:
   - **uploadFile()**: Sends a file to the `/api/storage/upload` endpoint using Fetch API.
   - **addFileToList(fileName)**: Adds a new file to the `fileList` when notified via WebSocket or during initial load.

### Notes:

- Replace `localhost:8080` with your actual application URL if deploying to a remote server.
- Ensure WebSocket (`/ws/{sessionId}`) and REST endpoints (`/api/storage/upload`, `/api/storage/files`, `/api/storage/download/{fileName}`) are correctly implemented in your Spring Boot application.
- Adjust styles and behaviors as per your application's design requirements.

This frontend will allow users to upload files, receive real-time notifications about new uploads via WebSocket, and dynamically list available files for download. Customize endpoints and behavior as needed based on your specific application requirements and architecture.

