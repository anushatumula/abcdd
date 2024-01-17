public class CmisControllerTest {

    @Mock
    private CmisUploadService cmisUploadService;

    @Mock
    private CmisDownloadService cmisDownloadService;

    @InjectMocks
    private CmisController cmisController;

    @Test
    public void testUploadFileToDocumentum_Success() throws Exception {
        MultipartFile mockMultipartFile = mock(MultipartFile.class);
        when(mockMultipartFile.getOriginalFilename()).thenReturn("test.txt");
        when(mockMultipartFile.getBytes()).thenReturn("file content".getBytes());

        when(cmisUploadService.uploadDocument(eq("test.txt"), eq("file content"))).thenReturn("Document uploaded successfully");

        ResponseEntity<String> response = cmisController.uploadFileToDocumentum(mockMultipartFile);

        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertEquals("Document uploaded successfully", response.getBody());

        verify(cmisUploadService).uploadDocument(eq("test.txt"), eq("file content"));
    }

    @Test
    public void testUploadFileToDocumentum_NullFile() {
        ResponseEntity<String> response = cmisController.uploadFileToDocumentum(null);

        assertEquals(HttpStatus.BAD_REQUEST, response.getStatusCode());
        assertEquals("Please upload multipartFile.", response.getBody());

        verifyNoInteractions(cmisUploadService);
    }

    @Test
    public void testUploadFileToDocumentum_Exception() throws Exception {
        MultipartFile mockMultipartFile = mock(MultipartFile.class);
        when(mockMultipartFile.getOriginalFilename()).thenReturn("test.txt");
        when(mockMultipartFile.getBytes()).thenThrow(new IOException("Simulated exception"));

        ResponseEntity<String> response = cmisController.uploadFileToDocumentum(mockMultipartFile);

        assertEquals(HttpStatus.BAD_REQUEST, response.getStatusCode());
        assertEquals("Error uploading file :: Simulated exception", response.getBody());

        verifyZeroInteractions(cmisUploadService);
    }

    // Similar tests for other methods in CmisController
    @Test

    public void testDownloadFileFromDocumentum_Success() throws Exception {
        String documentId = "0901ba8581313756";
        Document mockDocument = mock(Document.class);
        InputStream mockInputStream = new ByteArrayInputStream("file content".getBytes());

        when(cmisDownloadService.getDocumentById(documentId)).thenReturn(mockDocument);
        when(mockDocument.getContentStream()).thenReturn(mock(ContentStream.class));
        when(mockDocument.getContentStream().getStream()).thenReturn(mockInputStream);

        HttpServletResponse mockResponse = mock(HttpServletResponse.class);

        cmisController.downloadFileFromDocumentum(documentId, mockResponse);

        verify(mockResponse).setContentType(anyString());
        verify(mockResponse).setHeader(anyString(), anyString());
        verify(mockResponse.getOutputStream()).write(any(byte[].class));
        verify(mockResponse).flushBuffer();

        verify(cmisDownloadService).getDocumentById(documentId);
    }

    @Test
    public void testDownloadFileFromDocumentum_Exception() throws Exception {
        String documentId = "0901ba8581313756";

        when(cmisDownloadService.getDocumentById(documentId)).thenThrow(new RuntimeException("Simulated exception"));

        HttpServletResponse mockResponse = mock(HttpServletResponse.class);

        cmisController.downloadFileFromDocumentum(documentId, mockResponse);

        verify(mockResponse).setStatus(HttpStatus.INTERNAL_SERVER_ERROR.value());

        verify(cmisDownloadService).getDocumentById(documentId);
    }

    @Test
    public void testGetFilesUnderFolder_Success() throws Exception {
        String date = "MM_dd_yyyy";

        when(cmisDownloadService.getAllFilesInFolder(date)).thenReturn(Arrays.asList("file1.txt", "file2.txt"));

        ResponseEntity<?> response = cmisController.getFilesUnderFolder(date);

        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertEquals(Arrays.asList("file1.txt", "file2.txt"), response.getBody());

        verify(cmisDownloadService).getAllFilesInFolder(date);
    }

    @Test
    public void testGetFilesUnderFolder_Exception() throws Exception {
        String date = "MM_dd_yyyy";

        when(cmisDownloadService.getAllFilesInFolder(date)).thenThrow(new RuntimeException("Simulated exception"));

        ResponseEntity<?> response = cmisController.getFilesUnderFolder(date);

        assertEquals(HttpStatus.BAD_REQUEST, response.getStatusCode());
        assertEquals("Simulated exception", response.getBody());

        verify(cmisDownloadService).getAllFilesInFolder(date);
    }

    @Test
    public void testDeleteFileFromDocumentum_Success() throws Exception {
        String documentId = "0901ba8581313756";
        Document mockDocument = mock(Document.class);
        InputStream mockInputStream = new ByteArrayInputStream("file content".getBytes());

        when(cmisDownloadService.getDocumentById(documentId)).thenReturn(mockDocument);
        when(mockDocument.getContentStream()).thenReturn(mock(ContentStream.class));
        when(mockDocument.getContentStream().getStream()).thenReturn(mockInputStream);

        HttpServletResponse mockResponse = mock(HttpServletResponse.class);

        cmisController.deleteFileFromDocumentum(documentId, mockResponse);

        verify(mockResponse).setContentType(anyString());
        verify(mockResponse).setHeader(anyString(), anyString());
        verify(mockResponse.getOutputStream()).write(any(byte[].class));
        verify(mockResponse).flushBuffer();

        verify(mockDocument).delete();
        verify(cmisDownloadService).getDocumentById(documentId);
    }

    @Test
    public void testDeleteFileFromDocumentum_Exception() throws Exception {
        String documentId = "0901ba8581313756";

        when(cmisDownloadService.getDocumentById(documentId)).thenThrow(new RuntimeException("Simulated exception"));

        HttpServletResponse mockResponse = mock(HttpServletResponse.class);

        cmisController.deleteFileFromDocumentum(documentId, mockResponse);

        verify(mockResponse).setStatus(HttpStatus.INTERNAL_SERVER_ERROR.value());

        verifyZeroInteractions(mockResponse.getOutputStream());
        verifyZeroInteractions(mockResponse).write(any(byte[].class));
        verifyZeroInteractions(mockResponse).flushBuffer();

        verifyZeroInteractions(cmisDownloadService);
    }
	}
