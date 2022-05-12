USE WinSNAP

GO

-- YOU WILL HAVE TO GET THE REPORTS THAT MS LOWE SENDS TO MR MELLODY FIRST (EMAIL MS LOWE AND ASK HER FOR THESE), THEN RUN THE QUERY

--Note: The Erate report comes entirely from the edit check report (the POSSessSumm tablein WinSnap database)

-- to get the "reduced lunches, you have to add RMealsSumm + RMealsAcctCSumm. I had to use nullif to keep the query running. a zero was returned in one of the divisions, but all 131 school appear to be fine because of the nullif statement

SELECT "u_v_POSSumm"."SiteID" as Campus#, "Site"."Name" as CampusName, "OpSumm"."Enrollment" as TotalStudents, "OpSumm"."FreeCount" as FreeEligible, "u_v_POSSumm"."FMealsSumm" as FreeLunches, "OpSumm"."ReducedCount" as ReducedEligible, "u_v_POSSumm"."RMealsSumm" +"u_v_POSSumm"."RMealsAcctCSumm"  as ReducedLunches, "OpSumm"."FreeCount" + "OpSumm"."ReducedCount" as TotalEligible, "u_v_POSSumm"."FMealsSumm" + "u_v_POSSumm"."RMealsSumm" as Total_NSLP_Lunches, CAST(ROUND(("OpSumm"."FreeCount" + "OpSumm"."ReducedCount") /NULLIF( CONVERT(DECIMAL(8,4),"OpSumm"."Enrollment"),0),2) as numeric(36,2)) as EligiblePercent

INTO aaTempTable1

FROM   (("WinSNAP"."dbo"."u_v_POSSumm" "u_v_POSSumm" 

INNER JOIN "WinSNAP"."dbo"."Site" "Site" ON "u_v_POSSumm"."SiteID"="Site"."SiteID") 

INNER JOIN "WinSNAP"."dbo"."OpSumm" "OpSumm" ON ("u_v_POSSumm"."POSSessSummDate"="OpSumm"."DateTime") AND ("u_v_POSSumm"."SiteID"="OpSumm"."SiteID")) 

LEFT OUTER JOIN "WinSNAP"."dbo"."Provision2" "Provision2" ON "Site"."SiteID"="Provision2"."SiteID"

WHERE "u_v_POSSumm"."POSSessSummDate" = '2012-10-01'   

AND "u_v_POSSumm"."POSSession" = 'L' -- lunch is the only meal needed for the Erate report per Don Mellody

ORDER BY "u_v_POSSumm"."SiteID", "u_v_POSSumm"."POSSession", "u_v_POSSumm"."ProgramID"

--- i had to create a temptable to put in a percent sign

SELECT Campus#, CampusName, TotalStudents, FreeEligible, FreeLunches, ReducedEligible, ReducedLunches, TotalEligible, Total_NSLP_Lunches, CAST(EligiblePercent as varchar(100))+ '%' as EligiblePercent

INTO aaTempTable2

FROM aaTempTable1 

--i had to create another temp table to delete the spaces left of the percentage. the percentage is already rounded to the nearest hundreth in the first query

SELECT Campus#, CampusName, TotalStudents, FreeEligible, FreeLunches, ReducedEligible, ReducedLunches, TotalEligible, Total_NSLP_Lunches, RIGHT(EligiblePercent, 3) as EligiblePercent

FROM aaTempTable2 

--drop table aaTempTable1, aaTempTable2    this will delete (drop) the temp tables. just get the data from them and drop these afterwards

