package com.fidelity.ffio.guide.batchreports.service;

import static com.fidelity.ffio.guide.batchreports.utils.BatchReportsConstants.AUTOFEE_COLLECT;
import static com.fidelity.ffio.guide.batchreports.utils.BatchReportsConstants.NEW_LINE;
import static com.fidelity.ffio.guide.batchreports.utils.BatchReportsConstants.SPACE;
import static com.fidelity.ffio.guide.batchreports.utils.BatchReportsConstants.SUCCESS;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.math.BigDecimal;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;


import com.fidelity.fpcms.guide.model.GicsMoveFeesInfo;
import com.fidelity.fpcms.guide.model.ReportInfo;
import com.fidelity.fpcms.guide.model.ReportStatus;
import org.apache.commons.lang3.StringUtils;
import org.apache.commons.text.StringSubstitutor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import com.fidelity.ffio.guide.batchreports.mapper.AutofeeCollectionMapper;
import com.fidelity.ffio.guide.batchreports.utils.AppUtils;
import com.fidelity.ffio.guide.batchreports.utils.ApplicationException;;

@Component(AUTOFEE_COLLECT)

public class AutofeeCollectionReport implements BatchReportsCommand {

	private static final Logger LOG = LoggerFactory.getLogger(AutofeeCollectionReport.class);

	@Autowired
	AutofeeCollectionMapper autofeeCollectionMapper;
	
	List<String> messagesList = new ArrayList<>();
	public static final String TEMPLATE_DIRECTORY = "templates/";
	public static final String REPORT_NAME = "autofeeCollection";
	public static final String PLAN_ID_NBR = "plan_id_nbr";
	public static final String FEE_ACRU_A = "fee_acru_a";
	public static final String TXN_GL_ACC_PRST_NBR = "txn_gl_acc_prst_nbr";
	public static final int PAGE_SIZE = 36;
	
	public List<Map<String, Object>> getGicsMoveFeesInfo(Integer planFreqFeeParam, String fprsPlanParam) {
		LOG.info("AutoFee::getGicsMoveFeesInfo called, planFreqFee parameter is {} and fprsPlanParam is {} ", planFreqFeeParam, fprsPlanParam);
		return autofeeCollectionMapper.getGicsMoveFees(planFreqFeeParam, fprsPlanParam);
	}

	public List<Map<String, Object>> getFeeAcruInfo(int planIdNbr, String periodEndDate, String year) {
		LOG.info("AutoFee::getFeeAcruInfo called, for planIdNbr {}", planIdNbr);
		return autofeeCollectionMapper.getFeeAcru(periodEndDate, year);
	}

	public List<Map<String, Object>> getWrapProviderInfo() {
		LOG.info("AutoFee::getWrapProviderInfo called.");
		return autofeeCollectionMapper.getWrapProvider();
	}

	public List<Map<String, Object>> getMinStartDateInfo(String feeYrC) {
		LOG.info("AutoFee::getMinStartDateInfo called, current fee year is {} ", feeYrC);
		return autofeeCollectionMapper.getMinStartDate(feeYrC);
	}
	
	public int getDateDiffInfo(String minStartDate, String periodEndDate) {
		LOG.info("AutoFee::getDateDiffInfo called with minimum start date as {} and period end date {}", minStartDate, periodEndDate);
		return autofeeCollectionMapper.getDateDiff(minStartDate, periodEndDate);
	}

	public void updateLastSeqNbrInfo(int planIdNbr) {
		LOG.info("AutoFee:::updateLastSeqNbrInfo called for planIdNbr {} ", planIdNbr);
		autofeeCollectionMapper.updateLastSeqNbr();
		LOG.info("updateLastSeqNbrInfo: executed");
	}

	public int getNextBtchNbrInfo(int planIdNbr) {
		LOG.info("AutoFee::getNextBtchNbrInfo called for planIdNbr {} ", planIdNbr);
		return autofeeCollectionMapper.getNextBtchNbr();
	}

	public void updateTablesInfo(int btchNbr, int planIdNbr, int planWrapIdNbr, int feeCatIdNbr, int feeRecpIdNbr,
			String periodEndDate, String feeStrtDate, BigDecimal totalFeeAcruA, String txnDate) {
		LOG.info("AutoFee::getUpdateTablesInfo called for planIdNbr {} ", planIdNbr);
		autofeeCollectionMapper.updateTables(btchNbr, planIdNbr, planWrapIdNbr, feeCatIdNbr, feeRecpIdNbr,
				periodEndDate, feeStrtDate, totalFeeAcruA, txnDate);
	}

	public List<Map<String, Object>> getCurSTIFBalanceInfo() {
	    LOG.info("AutoFee::getCurSTIFBalanceInfo called.");
		return autofeeCollectionMapper.getCurSTIFBalance();
	}

	public List<Map<String, Object>> getPendFeePrimarySTIFInfo() {
		LOG.info("AutoFee::getPendFeePrimarySTIFInfo called.");
		return autofeeCollectionMapper.getPendFeePrimarySTIF();
	}

	public List<Map<String, Object>> getPendWirePrimarySTIFInfo() {
		LOG.info("AutoFee::getPendWirePrimarySTIFInfo called.");
		return autofeeCollectionMapper.getPendWirePrimarySTIF();
	}

	public List<Map<String, Object>> getPendRegRedemAInfo() {
		LOG.info("AutoFee::getPendRegRedemAInfo called.");
		return autofeeCollectionMapper.getPendRegRedemA();
	}

	public List<Map<String, Object>> getPendFeeRedemAInfo() {
		LOG.info("AutoFee::getPendFeeRedemAInfo called.");
		return autofeeCollectionMapper.getPendFeeRedemA();
	}

	public String getPeriodEndDateInfo() {
		LOG.info("AutoFee::getPeriodEndDate called.");
		return autofeeCollectionMapper.getPeriodEndDate();
	}

	public List<Map<String, Object>>  getFeeDetailTotalInfo(int planIdNbr, int feeCatIdNbr, int feeRecpIdNbr,
			String feeStrtDate, String periodEndDate) {
		LOG.info("AutoFee::getFeeDetailTotalInfo called for planIdNbr {}", planIdNbr);
		return autofeeCollectionMapper.getFeeDetailTotal(planIdNbr, feeCatIdNbr, feeRecpIdNbr, feeStrtDate, periodEndDate);
	}

	public String getCurrentYearInfo() {
		LOG.info("AutoFee::getCurrentYear called.");
		return autofeeCollectionMapper.getCurrentYear();
	}

	public String getTranscationDateInfo() {
		LOG.info("AutoFee::getTranscationDateInfo called.");
		return autofeeCollectionMapper.getTranscationDate();
	}

	public void executeFeeSumNewyearInfo() {
		LOG.info("AutoFee::getFeeSumNewyearInfo Stored Proc called.");
		autofeeCollectionMapper.executeFeeSumNewyear();
	}

	public void callGenAutoFeeTxnsInfo() {
		LOG.info("AutoFee::getCallGenAutoFeeTxnsInfo Stored Proc called.");
		autofeeCollectionMapper.callGenAutoFeeTxns();
	}
	
	public String determinePeriodEndDate(List<String> paramList) {
		if (StringUtils.isNotEmpty(paramList.get(0)) && !paramList.get(0).equals("NULL") ) {
					StringBuilder periodEndDateBuilder =  new StringBuilder();
					periodEndDateBuilder.append(paramList.get(0).substring(6, 10)).append("/")
								 .append(paramList.get(0).substring(0, 2)).append("/")
								 .append(paramList.get(0).substring(3, 5));
					return periodEndDateBuilder.toString();
				} else {
					return getPeriodEndDateInfo();
				}
			}

	public Integer planFreqFeeParam(String periodEndDate) {
		String planFreqFee = "";
		if (StringUtils.isNotEmpty(periodEndDate)) {
			String month = periodEndDate.substring(0, 2);
			
			if (month.equals("03") || month.equals("06") || month.equals("09")) {
				planFreqFee = "12,4";
			} else if (month.equals("12")) {
				planFreqFee = "12,4,1";
			} else {
				planFreqFee = "12";
			}
		}
		return Integer.valueOf(planFreqFee);
	}
	
	public String fprsPlanParam(String fprsPlanParam) {
		if (StringUtils.isNotEmpty(fprsPlanParam) && !fprsPlanParam.equals("NULL")) {
				return fprsPlanParam;
		}
		return "";
	}
	
	public BigDecimal executeGetFeeAcru(int btchNbr, String planTotals, GicsMoveFeesInfo gicsMovefeesInfo, String periodEndDate, 
			String year, BigDecimal totalFeeAcruA, int planWrapIdNbr, String transactionDate, BigDecimal feeAcruA) {
		int feeCatIdNbr = 0;
		int feeRecpIdNbr = 0;
		String feeStartDate = "";

		List<Map<String, Object>> feeAcruInfoList = getFeeAcruInfo(gicsMovefeesInfo.getPlanIdNbr(), periodEndDate, year);
		boolean feeDetailsFound = false;
			for (Map<String, Object> feeAcruInfo : feeAcruInfoList) {
				if ((int)feeAcruInfo.get(PLAN_ID_NBR) == gicsMovefeesInfo.getPlanIdNbr()) {
					feeCatIdNbr = (int) feeAcruInfo.get("fee_cat_id_nbr");
					feeRecpIdNbr = (int) feeAcruInfo.get("fee_recp_id_nbr");
					feeStartDate = (String) feeAcruInfo.get("fee_strt_date");
					feeDetailsFound = true;
				}
			}
			if(feeDetailsFound) {
				if (planTotals.equals("Y")) {
					totalFeeAcruA = executeGetFeeDetailTotal(gicsMovefeesInfo, feeCatIdNbr, feeRecpIdNbr, 
							feeStartDate, periodEndDate, feeAcruA, totalFeeAcruA);
				} else {
					totalFeeAcruA = BigDecimal.ZERO;
					totalFeeAcruA = executeGetFeeDetail(btchNbr, gicsMovefeesInfo, feeCatIdNbr, feeRecpIdNbr, 
							feeStartDate, periodEndDate, feeAcruA, totalFeeAcruA, planWrapIdNbr, transactionDate);
				}
			}
			return totalFeeAcruA;
	}
	
	public BigDecimal executeGetFeeDetailTotal(GicsMoveFeesInfo gicsMovefeesInfo, int feeCatIdNbr, int feeRecpIdNbr,
											   String feeStartDate, String periodEndDate, BigDecimal feeAcruA, BigDecimal totalFeeAcruA) {
		List<Map<String, Object>> feeDetailTotalInfoList = getFeeDetailTotalInfo(gicsMovefeesInfo.getPlanIdNbr(), feeCatIdNbr, 
				feeRecpIdNbr,feeStartDate, periodEndDate);
			for (Map<String, Object> feeDetailTotalInfo : feeDetailTotalInfoList) {
				if (feeDetailTotalInfo != null) {
					if (feeDetailTotalInfo.get(FEE_ACRU_A) != null) {
							feeAcruA =  BigDecimal.valueOf((Double)feeDetailTotalInfo.get(FEE_ACRU_A));
				}
			  }
			}
			totalFeeAcruA = totalFeeAcruA.add(feeAcruA);
			return totalFeeAcruA;
	}

	public BigDecimal executeGetFeeDetail(int btchNbr, GicsMoveFeesInfo gicsMovefeesInfo, int feeCatIdNbr, int feeRecpIdNbr,
			String feeStartDate, String periodEndDate, BigDecimal feeAcruA, BigDecimal totalFeeAcruA, int planWrapIdNbr, String transactionDate) {
		try {
		List<Map<String, Object>> feeDetailInfoList = getFeeDetailTotalInfo(gicsMovefeesInfo.getPlanIdNbr(), feeCatIdNbr, feeRecpIdNbr, 
				feeStartDate, periodEndDate);
		for (Map<String, Object> feeDetailInfo : feeDetailInfoList) {
			if (feeDetailInfo != null) {
				if (feeDetailInfo.get(FEE_ACRU_A) != null) {
					feeAcruA = BigDecimal.valueOf((Double)feeDetailInfo.get(FEE_ACRU_A));
				}
			 }
		  }
		totalFeeAcruA = feeAcruA;
		updateTables(gicsMovefeesInfo, feeCatIdNbr, feeRecpIdNbr, feeStartDate, periodEndDate, btchNbr, totalFeeAcruA, planWrapIdNbr, transactionDate);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return totalFeeAcruA;
	}
	
	public void updateTables(GicsMoveFeesInfo gicsMovefeesInfo, int feeCatIdNbr, int feeRecpIdNbr, 
			String feeStrtDate, String periodEndDate, int btchNbr, BigDecimal totalFeeAcruA, 
			int planWrapIdNbr, String txnDate) throws SQLException, ApplicationException {
		String fprsPlanC = StringUtils.rightPad(gicsMovefeesInfo.getFprsPlanC(), 8);
		String planLongNm = StringUtils.rightPad(gicsMovefeesInfo.getPlanLongNm(), 35);
		String processingMessage  = "Moving Fees to Request Process";
		messagesList.add(SPACE+fprsPlanC+planLongNm+processingMessage+NEW_LINE);
		updateTablesInfo(btchNbr, gicsMovefeesInfo.getPlanIdNbr(), planWrapIdNbr, feeCatIdNbr, feeRecpIdNbr,
				periodEndDate, feeStrtDate, totalFeeAcruA, txnDate);
	}
 	
	public List<GicsMoveFeesInfo> mapGicsMoveFeeToModel(List<Map<String, Object>> gicsMoveFeesListFrmDB) {
		GicsMoveFeesInfo gicsMoveFeesInfo;
		List<GicsMoveFeesInfo> gicsMoveFeesList = new ArrayList<GicsMoveFeesInfo>();
		for(Map<String, Object> gicsMoveFeesMap : gicsMoveFeesListFrmDB) {
			gicsMoveFeesInfo = new GicsMoveFeesInfo();
			gicsMoveFeesInfo.setFprsPlanC((String)gicsMoveFeesMap.get("fprs_plan_c"));
			gicsMoveFeesInfo.setGlAccPrstNbr((int)gicsMoveFeesMap.get("gl_acc_prst_nbr"));
			gicsMoveFeesInfo.setPlanIdNbr((int)gicsMoveFeesMap.get("plan_id_nbr"));
			gicsMoveFeesInfo.setPlanLongNm((String)gicsMoveFeesMap.get("plan_long_nm"));
			gicsMoveFeesList.add(gicsMoveFeesInfo);
			}
		return gicsMoveFeesList;
	}
	
	public int calPlanWrapId(int planWrapIdNbr, GicsMoveFeesInfo gicsMovefeesInfo, List<Map<String, Object>>  wrapProviderInfoList) {
		planWrapIdNbr = 0;
		for (Map<String, Object> wrapProviderInfo : wrapProviderInfoList ) { 
			 if ((int)wrapProviderInfo.get("wrap_plan_id_nbr") == gicsMovefeesInfo.getPlanIdNbr()) {
				  planWrapIdNbr = (int) wrapProviderInfo.get("plan_wrap_id_nbr");
			  }
		}
		return planWrapIdNbr;
	}
	
	public String calMinStartDate(String minStartDate, GicsMoveFeesInfo gicsMovefeesInfo, List<Map<String, Object>> minStartDateList) {
		for (Map<String, Object> minStartDateInfo: minStartDateList) {
			if((int) minStartDateInfo.get(PLAN_ID_NBR) == gicsMovefeesInfo.getPlanIdNbr()) {
				minStartDate = (String) minStartDateInfo.get("min_strt_date");
			}
		}
		return minStartDate;
	}
	
	public int calDateDiffInfo(int dateDiff, String minStartDate, String periodEndDate, 
			GicsMoveFeesInfo gicsMovefeesInfo, List<Map<String, Object>> minStartDateList) {
		for (Map<String, Object> minStartDateInfo : minStartDateList) {
			if((int) minStartDateInfo.get(PLAN_ID_NBR) == gicsMovefeesInfo.getPlanIdNbr()) {
				dateDiff = getDateDiffInfo(minStartDate, periodEndDate);
			}
		}
		return dateDiff;
	}
	
	public BigDecimal calCurStifBalA(BigDecimal curStifBalA, GicsMoveFeesInfo gicsMovefeesInfo, List<Map<String, Object>> curStifBalanceInfoList) {
		for (Map<String, Object> curStifBalanceInfo : curStifBalanceInfoList ) {
				if(curStifBalanceInfo.get("cur_stif_bal_a") != null) {
					if((int)curStifBalanceInfo.get(PLAN_ID_NBR) == gicsMovefeesInfo.getPlanIdNbr() && 
						(int)curStifBalanceInfo.get("gl_acc_nbr")	== gicsMovefeesInfo.getGlAccPrstNbr())
							curStifBalA = BigDecimal.valueOf((Double)curStifBalanceInfo.get("cur_stif_bal_a"));
				}
			}
		return curStifBalA;
	}
	
	public BigDecimal calPendFeeA(BigDecimal pendFeeA, List<Map<String, Object>> pendFeePrimarySTIFInfoList, GicsMoveFeesInfo gicsMovefeesInfo) {
		for (Map<String, Object> pendingFeePrimarySTIFInfo : pendFeePrimarySTIFInfoList ) {
			if((int)pendingFeePrimarySTIFInfo.get(PLAN_ID_NBR) == gicsMovefeesInfo.getPlanIdNbr() &&
					(int)pendingFeePrimarySTIFInfo.get(TXN_GL_ACC_PRST_NBR) == gicsMovefeesInfo.getGlAccPrstNbr()) {
				 	pendFeeA = BigDecimal.valueOf((Double) pendingFeePrimarySTIFInfo.get("pend_fee_a"));
				}
			}
		return pendFeeA;
	}
	
	public BigDecimal calPendWireA(BigDecimal pendWireA, List<Map<String, Object>> pendWirePrimarySTIFInfoList, GicsMoveFeesInfo gicsMovefeesInfo) {
		for (Map<String, Object> pendWirePrimarySTIFInfo : pendWirePrimarySTIFInfoList ) {
			if((int)pendWirePrimarySTIFInfo.get(PLAN_ID_NBR) == gicsMovefeesInfo.getPlanIdNbr()&&
				(int)pendWirePrimarySTIFInfo.get("gl_acc_1_nbr") == gicsMovefeesInfo.getGlAccPrstNbr()) {
					pendWireA = BigDecimal.valueOf((Float) pendWirePrimarySTIFInfo.get("pend_wire_a"));
				}
			}
		return pendWireA;
	}
	
	public BigDecimal calPendRegRedemA(BigDecimal pendRegRedemA, List<Map<String, Object>> pendRegRedemAInfoList, GicsMoveFeesInfo gicsMovefeesInfo) {
		for (Map<String, Object> pendRegRedemAInfo : pendRegRedemAInfoList ) {
			if((int)pendRegRedemAInfo.get(PLAN_ID_NBR) == gicsMovefeesInfo.getPlanIdNbr() &&
					(int)pendRegRedemAInfo.get(TXN_GL_ACC_PRST_NBR) == gicsMovefeesInfo.getGlAccPrstNbr()) {
					pendRegRedemA = BigDecimal.valueOf((Float) pendRegRedemAInfo.get("red_a"));
				}
			}
		return pendRegRedemA;
	}
	
	public BigDecimal calPendFeeRedemA(BigDecimal pendFeeRedemA, List<Map<String, Object>> pendFeeRedemAInfoList, GicsMoveFeesInfo gicsMovefeesInfo) {
		for (Map<String, Object> pendFeeRedemAInfo : pendFeeRedemAInfoList ) {
			
			if((int)pendFeeRedemAInfo.get(PLAN_ID_NBR) == gicsMovefeesInfo.getPlanIdNbr() &&
					(int)pendFeeRedemAInfo.get(TXN_GL_ACC_PRST_NBR) == gicsMovefeesInfo.getGlAccPrstNbr()) {
					pendFeeRedemA =  BigDecimal.valueOf((Float) pendFeeRedemAInfo.get("red_a2"));
				}
			}
		return pendFeeRedemA;
	}
	
	@Override
	public ReportStatus execute(ReportInfo reportInfo) throws ApplicationException {
		
		int dateDiff;
		int planWrapIdNbr = 0;
		String periodEndDate;
		String feeYrC;
		String transactionDate;
		String minStartDate = " ";
		BigDecimal curStifBalA = BigDecimal.ZERO;
		BigDecimal pendFeeA;
		BigDecimal pendWireA;
		BigDecimal pendRegRedemA;
		BigDecimal pendFeeRedemA;
		BigDecimal maxDedA;
		BigDecimal totalFeeAcruA;
		String planTotals;
		int btchNbr = 0;
		BigDecimal feeAcruA = BigDecimal.ZERO;
		
		try {
			periodEndDate = determinePeriodEndDate(reportInfo.getParamList());
			feeYrC = getCurrentYearInfo();
			transactionDate = getTranscationDateInfo();
			//calling stored-proc from the DB
			executeFeeSumNewyearInfo();
			List<Map<String, Object>> gicsMoveFeesListFrmDB = getGicsMoveFeesInfo(planFreqFeeParam(periodEndDate), fprsPlanParam(reportInfo.getParamList().get(1)));
			List<GicsMoveFeesInfo> gicsMoveFeesList =  mapGicsMoveFeeToModel(gicsMoveFeesListFrmDB);
			List<Map<String, Object>> wrapProviderInfoList = getWrapProviderInfo();
			List<Map<String, Object>> minStartDateList = getMinStartDateInfo(feeYrC);
			List<Map<String, Object>> curStifBalanceInfoList = getCurSTIFBalanceInfo();
			List<Map<String, Object>> pendFeePrimarySTIFInfoList = getPendFeePrimarySTIFInfo();
			List<Map<String, Object>> pendWirePrimarySTIFInfoList = getPendWirePrimarySTIFInfo();
			List<Map<String, Object>> pendRegRedemAInfoList = getPendRegRedemAInfo();
			List<Map<String, Object>> pendFeeRedemAInfoList = getPendFeeRedemAInfo();
		
			for (GicsMoveFeesInfo gicsMovefeesInfo : gicsMoveFeesList) {
				
				/*
				 * reset to 0 for every iteration of gicsMoveFeelist 
				 * list as this value has to be computed fresh for every call
				 */
				dateDiff = 0;
				pendFeeA = new BigDecimal(0);
				pendWireA = new BigDecimal(0);
				pendRegRedemA = new BigDecimal(0);
				pendFeeRedemA = new BigDecimal(0);
				maxDedA = new BigDecimal(0);
				totalFeeAcruA = new BigDecimal(0);
				String processingMessage;
				String fprsPlanC = StringUtils.rightPad(gicsMovefeesInfo.getFprsPlanC(), 8);
				String planLongNm = StringUtils.rightPad(gicsMovefeesInfo.getPlanLongNm(), 35);
								
				planWrapIdNbr = calPlanWrapId(planWrapIdNbr, gicsMovefeesInfo, wrapProviderInfoList);
				minStartDate = calMinStartDate(minStartDate, gicsMovefeesInfo, minStartDateList);
				dateDiff = calDateDiffInfo(dateDiff, minStartDate, periodEndDate, gicsMovefeesInfo, minStartDateList);
			    curStifBalA = calCurStifBalA(curStifBalA, gicsMovefeesInfo, curStifBalanceInfoList);
			    pendFeeA = calPendFeeA(pendFeeA, pendFeePrimarySTIFInfoList, gicsMovefeesInfo);
			    pendWireA = calPendWireA(pendWireA, pendWirePrimarySTIFInfoList, gicsMovefeesInfo);
			    pendRegRedemA = calPendRegRedemA(pendRegRedemA, pendRegRedemAInfoList, gicsMovefeesInfo);
			    pendFeeRedemA = calPendFeeRedemA(pendFeeRedemA, pendFeeRedemAInfoList, gicsMovefeesInfo);	
				/*
				 * For computing maxDedA 
				 * SQR had wrong naming for pendRegRedemA and pendFeeRedemA, verify if we need to subtract this
				 */
				maxDedA = (curStifBalA.subtract(pendFeeA).subtract(pendWireA));
				LOG.info("PLAN_ID_NBR: {}, fprsPlanC: {}", gicsMovefeesInfo.getPlanIdNbr(), fprsPlanC);
				LOG.info("curStifBalA: {}, pendFeeA: {}, pendWireA: {}, maxDedA: {}, dateDiff: {}", curStifBalA, pendFeeA, pendWireA, maxDedA, dateDiff);
							
			if (dateDiff < 0) {
				processingMessage = "Fees have already been collected for Plan";
				messagesList.add(SPACE+fprsPlanC+planLongNm+processingMessage+NEW_LINE);
			} else {
				planTotals = "Y";
				totalFeeAcruA = executeGetFeeAcru(btchNbr, planTotals, gicsMovefeesInfo, periodEndDate, feeYrC, totalFeeAcruA, planWrapIdNbr, transactionDate, feeAcruA);
				LOG.info("periodEndDate: {}, feeYrC: {}, totalFeeAcruA: {}, feeAcruA: {}", periodEndDate, feeYrC, totalFeeAcruA, feeAcruA);
				if (maxDedA.compareTo(totalFeeAcruA) == -1) {
					processingMessage = "STIF Balance < Fee Accrual";
					messagesList.add(SPACE+fprsPlanC+planLongNm+processingMessage+NEW_LINE);
				} else {
					if (totalFeeAcruA.compareTo(BigDecimal.ZERO) == 0) {
						processingMessage = "Cannot request fees totalling 0";
						messagesList.add(SPACE+fprsPlanC+planLongNm+processingMessage+NEW_LINE);
					} else {
						if(totalFeeAcruA.compareTo(BigDecimal.ZERO) == -1) {
						processingMessage = "Cannot request fees less than 0";
						messagesList.add(SPACE+fprsPlanC+planLongNm+processingMessage+NEW_LINE);
						} else {
							planTotals = "N";
							updateLastSeqNbrInfo(gicsMovefeesInfo.getPlanIdNbr());
							btchNbr = getNextBtchNbrInfo(gicsMovefeesInfo.getPlanIdNbr());
							executeGetFeeAcru(btchNbr, planTotals, gicsMovefeesInfo, periodEndDate, feeYrC, totalFeeAcruA, planWrapIdNbr, transactionDate, feeAcruA);
					}
				}
			}
		}
	}	
			//calling stored-proc from the DB
			callGenAutoFeeTxnsInfo();
			LOG.info("Message List size :::"+messagesList.size());
			buildAndPrintReport(messagesList, reportInfo, periodEndDate);
		} catch (Exception e) {
			e.printStackTrace();
			LOG.error("Exception when creating AutofeeCollectionReport", e);
			throw new ApplicationException("Exception when creating AutofeeCollectionReport", e);
		}
		return new ReportStatus(SUCCESS);
	}
	
	//TODO: Make this generic
	public void buildAndPrintReport(List<String> messagesList, ReportInfo reportInfo, String periodEndDate) throws IOException {
		LOG.info("AutoFee::called buildAndPrintReport messageList size is::: "+messagesList.size());
		List<String> fileContentsList = new ArrayList<>();
		int counter = 0;
		List<String> bufferList = null;
		int pageNumber = 1;
		while (counter < messagesList.size()) {
			bufferList = new ArrayList<>();
			for (int loopCounter = 0; loopCounter<PAGE_SIZE; loopCounter++) {
				if(counter < messagesList.size()) {
						bufferList.add(messagesList.get(counter));
						counter++;
					} else {
					break;
				}
			}
			fileContentsList = buildheaderMap(fileContentsList, String.valueOf(pageNumber), reportInfo, periodEndDate);
			fileContentsList = buildBodyMap(fileContentsList, bufferList, reportInfo);
			pageNumber++;	
		}
		fileContentsList.set(fileContentsList.size()-1, "\f");
		printReport(fileContentsList, reportInfo);	
	}
		
	public List<String> buildheaderMap(List<String> fileContentsList, String pageNumber, ReportInfo reportInfo, String periodEndDate) throws IOException {
		LOG.info("AutoFee::called buildheaderMap fileContentsList size is {} ::: ",fileContentsList.size());
		Map<String, Object> headerValuesMap = new HashMap<>();
		headerValuesMap.put("rptName", "AUTOFEE");
		headerValuesMap.put("pageNumber", pageNumber);
		headerValuesMap.put("currentDate", AppUtils.formatDate(new Date()));
		headerValuesMap.put("periodEndDate", periodEndDate);
		String templateFileName = TEMPLATE_DIRECTORY + REPORT_NAME + "-" + "header" + ".txt";
		if(Integer.parseInt(pageNumber) == 1) {
			fileContentsList.add(SPACE);
		}
		return processTemplate(fileContentsList, headerValuesMap, reportInfo, templateFileName);
	}
	
	public List<String> buildBodyMap(List<String> fileContentsList, List<String> messageContentList, ReportInfo reportInfo) throws IOException {
		LOG.info("AutoFee::called buildBodyMap fileContentsList size is {} ::: ",fileContentsList.size());
		Map<String, Object> bodyValuesMap = new HashMap<>();
		StringBuilder bodyString = new StringBuilder("");
		int counter = 1;
		for(String bodyLine: messageContentList) {
			if(counter == messageContentList.size()) {
				bodyString.append(StringUtils.chomp(bodyLine));
			} else {
				bodyString.append(bodyLine);
			}
			counter++;
		}
		bodyValuesMap.put("recordContent", bodyString.toString());
		String templateFileName = TEMPLATE_DIRECTORY + REPORT_NAME + "-" + "record" + ".txt";
		return processTemplate(fileContentsList, bodyValuesMap, reportInfo, templateFileName);
	}
	
	public List<String> processTemplate(List<String> fileContentsList, Map<String, Object> valuesMap, ReportInfo reportInfo, String templateFileName)
			throws IOException {
		LOG.info("AutoFee::called processTemplate fileContentsList size is {} ::: ",fileContentsList.size());
		ClassLoader classLoader = getClass().getClassLoader();
		InputStream in = classLoader.getResourceAsStream(templateFileName);
        BufferedReader br = new BufferedReader(new InputStreamReader(in));
		StringSubstitutor ss = new StringSubstitutor(valuesMap);
		String line = null;
		while ((line = br.readLine()) != null) {
				String lineReplaced = ss.replace(line);
				fileContentsList.add(lineReplaced);
			}
		LOG.info("AutoFee::returning from processTemplate fileContentsList size is {} ::: ",fileContentsList.size());
		return fileContentsList;
	}
	
	public void printReport(List<String> fileContentsList, ReportInfo reportInfo) throws IOException {
		LOG.info("AutoFee::called printReport for final printing,  fileContentsList size is {} ::: ",fileContentsList.size());
		String outputReportFile = reportInfo.getOutputReportFileName();
		Path path = Paths.get(outputReportFile);
		if (!fileContentsList.isEmpty())
			Files.write(path, fileContentsList, StandardCharsets.UTF_8);
	}
}
