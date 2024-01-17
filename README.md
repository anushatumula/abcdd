package com.fidelity.ffio.guide.batchreports.controller;

import com.fidelity.ffio.guide.batchreports.service.CmisDownloadService;
import com.fidelity.ffio.guide.batchreports.service.CmisUploadService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import org.apache.chemistry.opencmis.client.api.Document;
import org.apache.commons.io.IOUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.http.HttpServletResponse;
import java.io.InputStream;

@RestController
@RequestMapping("/cmis")
public class CmisController {

    public static final Logger logger = LoggerFactory.getLogger(CmisController.class);

    @Autowired
    private CmisUploadService cmisUploadService;

    @Autowired
    private CmisDownloadService cmisDownloadService;

    /**
     * Controller method to upload file to documentum.
     *
     * @param multipartFile
     * @return
     */
    @Operation(summary = "API to upload file to documentum")
    @ApiResponse(responseCode = "200", description = "Successfully uploaded the file to documentum")
    @PostMapping(value = "/upload", consumes = "multipart/form-data")
    public ResponseEntity<String> uploadFileToDocumentum(
            @Parameter(description = "File to be uploaded", content = @Content(mediaType = MediaType.MULTIPART_FORM_DATA_VALUE))
            @RequestParam(value = "fileToUpload", required = false) MultipartFile multipartFile) {
        try {
            if (multipartFile == null)
                return ResponseEntity.badRequest().body("Please upload multipartFile.");
            return ResponseEntity.ok(cmisUploadService.uploadDocument(multipartFile.getOriginalFilename(), new String(multipartFile.getBytes())));
        } catch (Exception e) {
            return ResponseEntity.badRequest().body("Error uploading file :: " + e.getMessage());
        }
    }

    /**
     * Controller method to download file based on document id.
     *
     * @param documentId
     * @param response
     */
    @Operation(summary = "To download file based on document id from documentum.")
    @GetMapping("/download")
    public void downloadFileFromDocumentum(
            @Parameter(description = "DocumentID to be downloaded (Eg. 0901ba8581313756)")
            @RequestParam("documentId") String documentId, HttpServletResponse response) {
        try {
            Document document = cmisDownloadService.getDocumentById(documentId);
            InputStream fileContent = document.getContentStream().getStream();
            response.setContentType(document.getContentStreamMimeType());
            response.setHeader(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + document.getName() + "\"");
            IOUtils.copy(fileContent, response.getOutputStream());
            response.flushBuffer();
        } catch (Exception e) {
            logger.warn("Error while getting file details from Documentum: {}", e.getMessage());
            response.setStatus(HttpStatus.INTERNAL_SERVER_ERROR.value());
        }
    }

    /**
     * Controller method to get all files details under reports folder.
     *
     * @param date
     * @return
     */
    @Operation(summary = "To get all files details under reports folder from documentum.")
    @GetMapping("/folder/files")
    public ResponseEntity<?> getFilesUnderFolder(
            @Parameter(description = "Date format MM_dd_yyyy")
            @RequestParam(value = "date", required = false) String date) {
        try {
            return new ResponseEntity<>(cmisDownloadService.getAllFilesInFolder(date), HttpStatus.OK);
        } catch (Exception e) {
            return ResponseEntity.badRequest().body(e.getMessage());
        }
    }

    /**
     * Controller method to download file based on document id.
     *
     * @param documentId
     * @param response
     */
    @Operation(summary = "To delete file based on document id from documentum.")
    @GetMapping("/delete")
    public void deleteFileFromDocumentum(
            @Parameter(description = "DocumentID to be downloaded (Eg. 0901ba8581313756)")
            @RequestParam("documentId") String documentId, HttpServletResponse response) {
        try {
            Document document = cmisDownloadService.getDocumentById(documentId);
            InputStream fileContent = document.getContentStream().getStream();
            response.setContentType(document.getContentStreamMimeType());
            response.setHeader(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + document.getName() + "\"");
            IOUtils.copy(fileContent, response.getOutputStream());
            response.flushBuffer();
            document.delete();
        } catch (Exception e) {
            logger.warn("Error while getting file details from Documentum: {}", e.getMessage());
            response.setStatus(HttpStatus.INTERNAL_SERVER_ERROR.value());
        }
    }
}
