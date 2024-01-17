  @Before
    public void setUp() {
        mockMessagesList = Arrays.asList("Message1", "Message2", "Message3");
        mockReportInfo = new ReportInfo("outputReport.txt");
    }

    @Test
    public void testDeterminePeriodEndDate() {
        List<String> paramList = Arrays.asList("01/15/2024");
        String result = autofeeCollectionReport.determinePeriodEndDate(paramList);
        assertEquals("2024/01/15", result);

        // Test with empty parameter
        paramList = new ArrayList<>();
        result = autofeeCollectionReport.determinePeriodEndDate(paramList);
        assertEquals(autofeeCollectionReport.getPeriodEndDateInfo(), result);
    }

    @Test
    public void testBuildAndPrintReport() throws IOException {
        // Mock the dependencies
        Logger mockLogger = mock(Logger.class);
        when(mockLogger.isInfoEnabled()).thenReturn(true);
        when(mockLogger.isErrorEnabled()).thenReturn(true);
        when(mockLogger.isDebugEnabled()).thenReturn(true);

        doNothing().when(mockLogger).info(anyString());
        doNothing().when(mockLogger).error(anyString(), any(Exception.class));

        when(autofeeCollectionReport.buildheaderMap(anyList(), anyString(), any(), anyString())).thenReturn(new ArrayList<>());
        when(autofeeCollectionReport.buildBodyMap(anyList(), anyList(), any())).thenReturn(new ArrayList<>());

        // Set the mock logger
        ReflectionTestUtils.setField(autofeeCollectionReport, "LOG", mockLogger);

        autofeeCollectionReport.buildAndPrintReport(mockMessagesList, mockReportInfo, "2024/01/15");

        // Add assertions based on the behavior you expect
        // For example, verify that the mockLogger.info method was called
        verify(mockLogger, times(1)).info(contains("AutoFee::called buildAndPrintReport messageList size is:::"));
    }

    @Test
    public void testBuildHeaderMap() throws IOException {
        List<String> fileContentsList = new ArrayList<>();
        String pageNumber = "1";
        String periodEndDate = "2024/01/15";
        when(autofeeCollectionReport.processTemplate(anyList(), anyMap(), any(), anyString())).thenReturn(new ArrayList<>());

        autofeeCollectionReport.buildheaderMap(fileContentsList, pageNumber, mockReportInfo, periodEndDate);

        // Add assertions based on the behavior you expect
    }

    @Test
    public void testBuildBodyMap() throws IOException {
        List<String> fileContentsList = new ArrayList<>();
        List<String> messageContentList = Arrays.asList("Content1", "Content2", "Content3");
        when(autofeeCollectionReport.processTemplate(anyList(), anyMap(), any(), anyString())).thenReturn(new ArrayList<>());

        autofeeCollectionReport.buildBodyMap(fileContentsList, messageContentList, mockReportInfo);

        // Add assertions based on the behavior you expect
    }

    @Test
    public void testProcessTemplate() throws IOException {
        List<String> fileContentsList = new ArrayList<>();
        Map<String, Object> valuesMap = new HashMap<>();
        String templateFileName = "templates/autofeeCollection-header.txt";

        autofeeCollectionReport.processTemplate(fileContentsList, valuesMap, mockReportInfo, templateFileName);

        // Add assertions based on the behavior you expect
    }

    @Test
    public void testPrintReport() throws IOException {
        List<String> fileContentsList = Arrays.asList("Line1", "Line2", "Line3");
        when(mockReportInfo.getOutputReportFileName()).thenReturn("outputReport.txt");

        autofeeCollectionReport.printReport(fileContentsList, mockReportInfo);

        // Add assertions based on the behavior you expect
        Path path = Paths.get("outputReport.txt");
        assertTrue(Files.exists(path));

        // Clean up: Delete the created file after the test
        Files.delete(path);
    }
	
	 @Test
    public void testCalPlanWrapId() {
        GicsMoveFeesInfo gicsMovefeesInfo = new GicsMoveFeesInfo();
        List<Map<String, Object>> wrapProviderInfoList = new ArrayList<>();
        wrapProviderInfoList.add(Map.of("wrap_plan_id_nbr", 1, "plan_wrap_id_nbr", 100));
        Mockito.when(autofeeCollectionMapper.someMethod()).thenReturn(/* some value */);

        int result = autofeeCollectionReport.calPlanWrapId(0, gicsMovefeesInfo, wrapProviderInfoList);

        assertEquals(0, result); // Add the expected result based on your logic
    }

     @Test
    public void testCalMinStartDate() {
        // Prepare test data
        GicsMoveFeesInfo gicsMoveFeesInfo = new GicsMoveFeesInfo();
        List<Map<String, Object>> minStartDateList = new ArrayList<>();
        minStartDateList.add(Collections.singletonMap("plan_id_nbr", 789));

        // Mock behavior
        when(autofeeCollectionMapper.getMinStartDateList()).thenReturn(minStartDateList);

        // Call the method
        String result = autofeeCollectionReport.calMinStartDate("", gicsMoveFeesInfo, minStartDateList);

        // Assert the result
        assertEquals("", result);
    }

    @Test
    public void testCalDateDiffInfo() {
        // Prepare test data
        GicsMoveFeesInfo gicsMoveFeesInfo = new GicsMoveFeesInfo();
        List<Map<String, Object>> minStartDateList = new ArrayList<>();
        minStartDateList.add(Collections.singletonMap("plan_id_nbr", 999));

        // Mock behavior
        when(autofeeCollectionMapper.getMinStartDateList()).thenReturn(minStartDateList);

        // Call the method
        int result = autofeeCollectionReport.calDateDiffInfo(0, "2022-01-01", "2022-01-31", gicsMoveFeesInfo, minStartDateList);

        // Assert the result
        assertEquals(30, result);
    }

    // Add similar tests for other methods...

  @Test
    public void testCalPlanWrapId() {
        // Prepare test data
        GicsMoveFeesInfo gicsMoveFeesInfo = new GicsMoveFeesInfo();
        List<Map<String, Object>> wrapProviderInfoList = new ArrayList<>();
        wrapProviderInfoList.add(Collections.singletonMap("wrap_plan_id_nbr", 123));

        // Mock behavior
        when(autofeeCollectionMapper.getWrapProviderInfoList()).thenReturn(wrapProviderInfoList);

        // Call the method
        int result = autofeeCollectionReport.calPlanWrapId(0, gicsMoveFeesInfo, wrapProviderInfoList);

        // Assert the result
        assertEquals(123, result);
    }

    // Similar tests for other methods...

    @Test
    public void testCalPendFeeRedemA() {
        // Prepare test data
        GicsMoveFeesInfo gicsMoveFeesInfo = new GicsMoveFeesInfo();
        List<Map<String, Object>> pendFeeRedemAInfoList = new ArrayList<>();
        pendFeeRedemAInfoList.add(Collections.singletonMap("plan_id_nbr", 456));

        // Mock behavior
        when(autofeeCollectionMapper.getPendFeeRedemAInfoList()).thenReturn(pendFeeRedemAInfoList);

        // Call the method
        BigDecimal result = autofeeCollectionReport.calPendFeeRedemA(BigDecimal.ZERO, pendFeeRedemAInfoList, gicsMoveFeesInfo);

        // Assert the result
        assertEquals(BigDecimal.ZERO, result);
    }
    @Test
    public void testCalCurStifBalA() {
        // Prepare test data
        GicsMoveFeesInfo gicsMoveFeesInfo = new GicsMoveFeesInfo();
        List<Map<String, Object>> curStifBalanceInfoList = new ArrayList<>();
        curStifBalanceInfoList.add(Collections.singletonMap("plan_id_nbr", 555));

        // Mock behavior
        when(autofeeCollectionMapper.getCurStifBalanceInfoList()).thenReturn(curStifBalanceInfoList);

        // Call the method
        BigDecimal result = autofeeCollectionReport.calCurStifBalA(BigDecimal.ZERO, gicsMoveFeesInfo, curStifBalanceInfoList);

        // Assert the result
        assertEquals(BigDecimal.ZERO, result);
    }

    // Add similar tests for other methods...

    @Test
    public void testCalPendRegRedemA() {
        // Prepare test data
        GicsMoveFeesInfo gicsMoveFeesInfo = new GicsMoveFeesInfo();
        List<Map<String, Object>> pendRegRedemAInfoList = new ArrayList<>();
        pendRegRedemAInfoList.add(Collections.singletonMap("plan_id_nbr", 777));

        // Mock behavior
        when(autofeeCollectionMapper.getPendRegRedemAInfoList()).thenReturn(pendRegRedemAInfoList);

        // Call the method
        BigDecimal result = autofeeCollectionReport.calPendRegRedemA(BigDecimal.ZERO, pendRegRedemAInfoList, gicsMoveFeesInfo);

        // Assert the result
        assertEquals(BigDecimal.ZERO, result);
    }

    // Add similar tests for other methods...

    @Test
    public void testCalPendFeeRedemA() {
        // Prepare test data
        GicsMoveFeesInfo gicsMoveFeesInfo = new GicsMoveFeesInfo();
        List<Map<String, Object>> pendFeeRedemAInfoList = new ArrayList<>();
        pendFeeRedemAInfoList.add(Collections.singletonMap("plan_id_nbr", 888));

        // Mock behavior
        when(autofeeCollectionMapper.getPendFeeRedemAInfoList()).thenReturn(pendFeeRedemAInfoList);

        // Call the method
        BigDecimal result = autofeeCollectionReport.calPendFeeRedemA(BigDecimal.ZERO, pendFeeRedemAInfoList, gicsMoveFeesInfo);

        // Assert the result
        assertEquals(BigDecimal.ZERO, result);
    }
	
	 @Test
    void testFprsPlanParam() {
        // Test when fprsPlanParam is not NULL
        String result = autofeeCollectionReport.fprsPlanParam("test");
        assertEquals("test", result);

        // Test when fprsPlanParam is NULL
        result = autofeeCollectionReport.fprsPlanParam(null);
        assertEquals("", result);
    }

    // Add similar tests for other methods

    // Example test for executeGetFeeAcru method
    @Test
    void testExecuteGetFeeAcru() {
        GicsMoveFeesInfo gicsMoveFeesInfo = new GicsMoveFeesInfo();
        // Mocking getFeeAcruInfo method
        when(autofeeCollectionMapper.getFeeAcruInfo(anyInt(), anyString(), anyString()))
                .thenReturn(List.of(Map.of("plan_id_nbr", 1, "fee_cat_id_nbr", 2, "fee_recp_id_nbr", 3, "fee_strt_date", "2022-01-01")));

        BigDecimal result = autofeeCollectionReport.executeGetFeeAcru(1, "Y", gicsMoveFeesInfo, "2022-01-01", "2022", BigDecimal.ZERO, 1, "2022-01-01", BigDecimal.ZERO);

        assertEquals(BigDecimal.ZERO, result); // Add your expected value
        // Verify that getFeeAcruInfo method was called with the expected parameters
        verify(autofeeCollectionMapper, times(1)).getFeeAcruInfo(anyInt(), anyString(), anyString());
    }

    // Add similar tests for other methods

    // Example test for updateTables method
    @Test
    void testUpdateTables() throws SQLException, ApplicationException {
        GicsMoveFeesInfo gicsMoveFeesInfo = new GicsMoveFeesInfo();
        gicsMoveFeesInfo.setFprsPlanC("testPlan");
        gicsMoveFeesInfo.setPlanLongNm("Test Plan");
        when(autofeeCollectionMapper.updateTablesInfo(anyInt(), anyInt(), anyInt(), anyInt(), anyInt(),
                anyString(), anyString(), any(BigDecimal.class), anyString())).thenReturn(1);

        assertDoesNotThrow(() -> {
            autofeeCollectionReport.updateTables(gicsMoveFeesInfo, 1, 2, "2022-01-01", "2022-01-01", 1, BigDecimal.ZERO, 1, "2022-01-01");
        });

        // Verify that updateTablesInfo method was called with the expected parameters
        verify(autofeeCollectionMapper, times(1)).updateTablesInfo(anyInt(), anyInt(), anyInt(), anyInt(), anyInt(),
                anyString(), anyString(), any(BigDecimal.class), anyString());
    }
	
	 @Test
    void testFprsPlanParam() {
        // Test when fprsPlanParam is not NULL
        String result = autofeeCollectionReport.fprsPlanParam("test");
        assertEquals("test", result);

        // Test when fprsPlanParam is NULL
        result = autofeeCollectionReport.fprsPlanParam(null);
        assertEquals("", result);
    }

    @Test
    void testExecuteGetFeeAcru() {
        GicsMoveFeesInfo gicsMoveFeesInfo = new GicsMoveFeesInfo();
        // Mocking getFeeAcruInfo method
        when(autofeeCollectionMapper.getFeeAcruInfo(anyInt(), anyString(), anyString()))
                .thenReturn(List.of(Map.of("plan_id_nbr", 1, "fee_cat_id_nbr", 2, "fee_recp_id_nbr", 3, "fee_strt_date", "2022-01-01")));

        BigDecimal result = autofeeCollectionReport.executeGetFeeAcru(1, "Y", gicsMoveFeesInfo, "2022-01-01", "2022", BigDecimal.ZERO, 1, "2022-01-01", BigDecimal.ZERO);

        assertEquals(BigDecimal.ZERO, result); // Add your expected value
        // Verify that getFeeAcruInfo method was called with the expected parameters
        verify(autofeeCollectionMapper, times(1)).getFeeAcruInfo(anyInt(), anyString(), anyString());
    }

    @Test
    void testExecuteGetFeeDetailTotal() {
        GicsMoveFeesInfo gicsMoveFeesInfo = new GicsMoveFeesInfo();
        // Mocking getFeeDetailTotalInfo method
        when(autofeeCollectionMapper.getFeeDetailTotalInfo(anyInt(), anyInt(), anyInt(), anyString(), anyString()))
                .thenReturn(List.of(Map.of("fee_acru_a", 100.0)));

        BigDecimal result = autofeeCollectionReport.executeGetFeeDetailTotal(gicsMoveFeesInfo, 1, 2, "2022-01-01", "2022-01-31", BigDecimal.ZERO, BigDecimal.ZERO);

        assertEquals(BigDecimal.valueOf(100.0), result); // Add your expected value
        // Verify that getFeeDetailTotalInfo method was called with the expected parameters
        verify(autofeeCollectionMapper, times(1)).getFeeDetailTotalInfo(anyInt(), anyInt(), anyInt(), anyString(), anyString());
    }

    @Test
    void testExecuteGetFeeDetail() {
        GicsMoveFeesInfo gicsMoveFeesInfo = new GicsMoveFeesInfo();
        // Mocking getFeeDetailTotalInfo method
        when(autofeeCollectionMapper.getFeeDetailTotalInfo(anyInt(), anyInt(), anyInt(), anyString(), anyString()))
                .thenReturn(List.of(Map.of("fee_acru_a", 100.0)));

        BigDecimal result = autofeeCollectionReport.executeGetFeeDetail(1, gicsMoveFeesInfo, 2, 3, "2022-01-01", "2022-01-31", BigDecimal.ZERO, BigDecimal.ZERO, 4, "2022-01-01");

        assertEquals(BigDecimal.valueOf(100.0), result); // Add your expected value
        // Verify that getFeeDetailTotalInfo method was called with the expected parameters
        verify(autofeeCollectionMapper, times(1)).getFeeDetailTotalInfo(anyInt(), anyInt(), anyInt(), anyString(), anyString());
        // Add additional verification for updateTables method
        verify(autofeeCollectionMapper, times(1)).updateTablesInfo(anyInt(), anyInt(), anyInt(), anyInt(), anyInt(),
                anyString(), anyString(), any(BigDecimal.class), anyString());
    }

    @Test
    void testUpdateTables() throws SQLException, ApplicationException {
        GicsMoveFeesInfo gicsMoveFeesInfo = new GicsMoveFeesInfo();
        gicsMoveFeesInfo.setFprsPlanC("testPlan");
        gicsMoveFeesInfo.setPlanLongNm("Test Plan");
        when(autofeeCollectionMapper.updateTablesInfo(anyInt(), anyInt(), anyInt(), anyInt(), anyInt(),
                anyString(), anyString(), any(BigDecimal.class), anyString())).thenReturn(1);

        assertDoesNotThrow(() -> {
            autofeeCollectionReport.updateTables(gicsMoveFeesInfo, 1, 2, "2022-01-01", "2022-01-01", 1, BigDecimal.ZERO, 1, "2022-01-01");
        });

        // Verify that updateTablesInfo method was called with the expected parameters
        verify(autofeeCollectionMapper, times(1)).updateTablesInfo(anyInt(), anyInt(), anyInt(), anyInt(), anyInt(),
                anyString(), anyString(), any(BigDecimal.class), anyString());
    }

    @Test
    void testMapGicsMoveFeeToModel() {
        // Mocking data from the database
        List<Map<String, Object>> gicsMoveFeesListFrmDB = List.of(
                Map.of("fprs_plan_c", "testPlan", "gl_acc_prst_nbr", 1, "plan_id_nbr", 2, "plan_long_nm", "Test Plan"));

        List<GicsMoveFeesInfo> result = autofeeCollectionReport.mapGicsMoveFeeToModel(gicsMoveFeesListFrmDB);

        assertEquals(1, result.size()); // Add your expected value
        // Add more assertions based on the data you expect
        assertEquals("testPlan", result.get(0).getFprsPlanC());
        assertEquals(1, result.get(0).getGlAccPrstNbr());
        assertEquals(2, result.get(0).getPlanIdNbr());
        assertEquals("Test Plan", result.get(0).getPlanLongNm());
    }
	
	  @BeforeEach
    void setUp() {
        MockitoAnnotations.initMocks(this);
    }

    @Test
    void testDeterminePeriodEndDate() {
        List<String> paramList = Arrays.asList("01/15/2024");
        String result = autofeeCollectionReport.determinePeriodEndDate(paramList);
        assertEquals("2024/01/15", result);

        // Test for empty paramList
        paramList = Arrays.asList();
        result = autofeeCollectionReport.determinePeriodEndDate(paramList);
        assertEquals(autofeeCollectionReport.getPeriodEndDateInfo(), result);

        // Test for NULL in paramList
        paramList = Arrays.asList("NULL");
        result = autofeeCollectionReport.determinePeriodEndDate(paramList);
        assertEquals(autofeeCollectionReport.getPeriodEndDateInfo(), result);
    }

    @Test
    void testPlanFreqFeeParam() {
        // Test for a specific month
        String periodEndDate = "2024/01/15";
        Integer result = autofeeCollectionReport.planFreqFeeParam(periodEndDate);
        assertEquals(Integer.valueOf("12"), result);

        // Test for a different month
        periodEndDate = "2024/03/15";
        result = autofeeCollectionReport.planFreqFeeParam(periodEndDate);
        assertEquals(Integer.valueOf("12,4"), result);

        // Add more tests for other scenarios
    }

    @Test
    void testExecute() {
        // Mock necessary dependencies
        when(autofeeCollectionMapper.getGicsMoveFeesInfo(anyInt(), anyString())).thenReturn(someList);
        // Mock other method calls as needed

        // Create a ReportInfo instance with required parameters
        ReportInfo reportInfo = new ReportInfo();
        reportInfo.setParamList(Arrays.asList("01/15/2024", "someValue"));

        // Perform the test
        ReportStatus result = autofeeCollectionReport.execute(reportInfo);

        // Verify the expected behavior, for example:
        assertEquals(ReportStatus.SUCCESS, result.getStatus());
        // Add more verification as needed

        // Verify that specific methods were called with the expected parameters
        verify(autofeeCollectionMapper).getGicsMoveFeesInfo(anyInt(), anyString());
        // Verify other method calls as needed
    }

    //
