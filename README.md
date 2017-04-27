# Attendance-Quality-Tracker-
Queries for attendance quality tracker 

```
-----------------------MISMATCH BETWEEN DL AND PS ATTENDANCE CODES

SELECT
DL_ATT.STUDENTSCHOOLID AS STUDENT_NUMBER,
DL_ATT.SCHOOLNAME AS SCHOOL_NAME,
--SC.ABBREVIATION,
'DL: '+DL_ATT.BEHAVIOR+' / '+'PS: '+PS_ATT.[DESCRIPTION] AS ERROR_GROUP, --mismatched attendance values in DL and PS
CAST(DL_ATT.BEHAVIORDATE AS DATE) AS ERROR_DATE,
PS_ATT.YEARID AS YEARID,
S.GRADE_LEVEL AS GRADE_LEVEL,
S.LASTFIRST AS LASTFIRST,
'Mismatch between Deanslist and Powerschool attendance records' AS ERROR
FROM CUSTOM.CUSTOM_DLATTENDANCE_RAW DL_ATT
JOIN POWERSCHOOL.POWERSCHOOL_STUDENTS S ON S.STUDENT_NUMBER = DL_ATT.STUDENTSCHOOLID
JOIN POWERSCHOOL.POWERSCHOOL_SCHOOLS SC ON SC.SCHOOL_NUMBER = S.SCHOOLID
JOIN (SELECT
		S.STUDENT_NUMBER STUDENT_NUMBER,
		CD.DATE_VALUE,
		E.ENTRYDATE,
		E.EXITDATE,
		A.YEARID,
		AC.ATT_CODE,
		CASE
			WHEN CD.DATE_VALUE>=E.ENTRYDATE AND CD.DATE_VALUE<E.EXITDATE THEN 1
			ELSE 0
		END ENROLL_STATUS,
		COALESCE(AC.DESCRIPTION,'Present') "DESCRIPTION"
		FROM POWERSCHOOL.POWERSCHOOL_SCHOOLS SC
		JOIN (
		  SELECT ID STUDENTID, SCHOOLID, GRADE_LEVEL, ENTRYDATE, ENTRYCODE, EXITDATE, EXITCODE FROM POWERSCHOOL.POWERSCHOOL_STUDENTS
		  UNION ALL
		  SELECT STUDENTID, SCHOOLID, GRADE_LEVEL, ENTRYDATE, ENTRYCODE, EXITDATE, EXITCODE FROM POWERSCHOOL.POWERSCHOOL_REENROLLMENTS
		) E ON E.SCHOOLID = SC.SCHOOL_NUMBER AND SC.SCHOOL_NUMBER!=2001
		JOIN POWERSCHOOL.POWERSCHOOL_STUDENTS S ON S.ID = E.STUDENTID
		JOIN POWERSCHOOL.POWERSCHOOL_CALENDAR_DAY CD ON CD.SCHOOLID = E.SCHOOLID AND CD.DATE_VALUE BETWEEN E.ENTRYDATE AND E.EXITDATE-1 AND CD.INSESSION = 1
		LEFT OUTER JOIN POWERSCHOOL.POWERSCHOOL_ATTENDANCE A ON A.STUDENTID = E.STUDENTID AND A.ATT_DATE = CD.DATE_VALUE AND ATT_MODE_CODE = 'ATT_MODEDAILY'
		LEFT JOIN POWERSCHOOL.POWERSCHOOL_ATTENDANCE_CODE AC ON AC.ID = A.ATTENDANCE_CODEID
		--WHERE CD.DATE_VALUE >= '2015-08-10'
		) PS_ATT ON PS_ATT.STUDENT_NUMBER = DL_ATT.STUDENTSCHOOLID AND PS_ATT.DATE_VALUE = DL_ATT.BEHAVIORDATE
WHERE BEHAVIORCATEGORY = 'Daily Attendance'
--AND DL_ATT.BEHAVIOR NOT LIKE '%'+PS_ATT.[DESCRIPTION]
AND PS_ATT.ATT_CODE != DL_ATT.ATTENDANCECODE
AND PS_ATT.ENROLL_STATUS = 1
AND S.HOME_ROOM NOT LIKE 'LC%'

----------------MISMATCH BETWEEN DL AND PS ATTENDANCE CODES (Compared to Present)
SELECT
DL_ATT.STUDENTSCHOOLID AS STUDENT_NUMBER,
DL_ATT.SCHOOLNAME AS SCHOOL_NAME,
--SC.ABBREVIATION,
'DL: '+DL_ATT.BEHAVIOR+' / '+'PS: '+PS_ATT.[DESCRIPTION] AS ERROR_GROUP, --mismatched attendance values in DL and PS
CAST(DL_ATT.BEHAVIORDATE AS DATE) AS ERROR_DATE
,PS_ATT.YEARID AS YEARID,
S.GRADE_LEVEL AS GRADE_LEVEL,
S.LASTFIRST AS LASTFIRST,
'Mismatch between Deanslist and Powerschool attendance records when PS is Present' AS ERROR
FROM CUSTOM.CUSTOM_DLATTENDANCE_RAW DL_ATT
JOIN POWERSCHOOL.POWERSCHOOL_STUDENTS S ON S.STUDENT_NUMBER = DL_ATT.STUDENTSCHOOLID
JOIN POWERSCHOOL.POWERSCHOOL_SCHOOLS SC ON SC.SCHOOL_NUMBER = S.SCHOOLID
JOIN (SELECT
		S.STUDENT_NUMBER STUDENT_NUMBER,
		CD.DATE_VALUE,
		E.ENTRYDATE,
		E.EXITDATE,
		A.YEARID,
		AC.ATT_CODE,
		CASE
			WHEN CD.DATE_VALUE>=E.ENTRYDATE AND CD.DATE_VALUE<E.EXITDATE THEN 1
			ELSE 0
		END ENROLL_STATUS,
		COALESCE(AC.DESCRIPTION,'Present') "DESCRIPTION"
		FROM POWERSCHOOL.POWERSCHOOL_SCHOOLS SC
		JOIN (
		  SELECT ID STUDENTID, SCHOOLID, GRADE_LEVEL, ENTRYDATE, ENTRYCODE, EXITDATE, EXITCODE FROM POWERSCHOOL.POWERSCHOOL_STUDENTS
		  UNION ALL
		  SELECT STUDENTID, SCHOOLID, GRADE_LEVEL, ENTRYDATE, ENTRYCODE, EXITDATE, EXITCODE FROM POWERSCHOOL.POWERSCHOOL_REENROLLMENTS
		) E ON E.SCHOOLID = SC.SCHOOL_NUMBER AND SC.SCHOOL_NUMBER!=2001
		JOIN POWERSCHOOL.POWERSCHOOL_STUDENTS S ON S.ID = E.STUDENTID
		JOIN POWERSCHOOL.POWERSCHOOL_CALENDAR_DAY CD ON CD.SCHOOLID = E.SCHOOLID AND CD.DATE_VALUE BETWEEN E.ENTRYDATE AND E.EXITDATE-1 AND CD.INSESSION = 1
		LEFT OUTER JOIN POWERSCHOOL.POWERSCHOOL_ATTENDANCE A ON A.STUDENTID = E.STUDENTID AND A.ATT_DATE = CD.DATE_VALUE AND ATT_MODE_CODE = 'ATT_MODEDAILY'
		LEFT JOIN POWERSCHOOL.POWERSCHOOL_ATTENDANCE_CODE AC ON AC.ID = A.ATTENDANCE_CODEID
		--WHERE CD.DATE_VALUE >= '2015-08-10'
		) PS_ATT ON PS_ATT.STUDENT_NUMBER = DL_ATT.STUDENTSCHOOLID AND PS_ATT.DATE_VALUE = DL_ATT.BEHAVIORDATE
WHERE BEHAVIORCATEGORY = 'Daily Attendance'
AND PS_ATT.ATT_CODE IS NULL 
AND DL_ATT.BEHAVIOR != 'Present'
AND DL_ATT.BEHAVIOR != 'No Attendance Taken'
AND DL_ATT.STUDENTSCHOOLID != 10075
--AND PS_ATT.YEARID = 26 
AND PS_ATT.ENROLL_STATUS = 1
AND DL_ATT.BEHAVIORDATE > '2016-08-08'

-----------Data in DL Raw Attendance Table?
select *
from custom.custom_DLAttendance_raw
where 
BehaviorDate = dateadd(day,datediff(day,1,GETDATE()),0)
AND BehaviorCategory = 'Daily Attendance'

-------Number of Present Attendance Records
SELECT 
A.STUDENTID AS STUDENT_NUMBER,
SCH.ABBREVIATION AS SCHOOL_NAME,
AC.PRESENCE_STATUS_CD AS ATTENDANCE_STATUS,
A.ATT_DATE AS DATE,
A.YEARID
FROM POWERSCHOOL.POWERSCHOOL_ATTENDANCE A
JOIN POWERSCHOOL.POWERSCHOOL_SCHOOLS SCH ON SCH.SCHOOL_NUMBER = A.SCHOOLID
JOIN POWERSCHOOL.POWERSCHOOL_ATTENDANCE_CODE AC ON AC.SCHOOLID = A.SCHOOLID
WHERE ATT_DATE = dateadd(day,datediff(day,1,GETDATE()),0)
AND A.YEARID = 26
AND AC.PRESENCE_STATUS_CD = 'Present' 


------Number of Present Records
SELECT 

  s.student_number AS StudentNumber
, s.id AS PSID
, s.lastfirst AS FullName
, sc.abbreviation AS SchoolName
, dsc.schoolyear4digit AS SchoolYear4Digit
, cd.date_value AS FullDate
, cd.membershipvalue AS Membership
, cd.insession AS InSession
, COALESCE(ac.att_code,'P') AttendanceCode
, COALESCE(ac.description,'Present') AttendanceDescription
, COALESCE(ac.presence_status_cd,'Present') PresentStatus
,s.entrydate as entrydate
,s.exitdate as exitdate

FROM [powerschool].[powerschool_students] s
INNER JOIN [powerschool].[PowerSchool_CALENDAR_DAY] cd ON (cd.date_value BETWEEN s.entrydate AND s.exitdate-1) AND s.schoolid = cd.schoolid
LEFT JOIN [powerschool].[PowerSchool_ATTENDANCE] att ON (att.studentid = s.id AND att.att_date = cd.date_value AND att.att_mode_code = 'ATT_ModeDaily')
LEFT JOIN [powerschool].[PowerSchool_ATTENDANCE_CODE] ac ON ac.id = att.attendance_codeid
INNER JOIN [powerschool].[PowerSchool_SCHOOLS] sc ON sc.school_number = s.schoolid
INNER JOIN [custom].[custom_StudentBridge] sb ON sb.student_number = s.student_number
INNER JOIN [custom].[custom_Students] cs ON cs.systemstudentid = sb.systemstudentid
INNER JOIN [dw].[DW_dimSchoolCalendar] dsc ON (dsc.systemschoolid = CONVERT(varchar,s.schoolid)  AND dsc.fulldate = cd.date_value)


WHERE 1=1
AND cd.insession = 1
AND cd.date_value = dateadd(day,datediff(day,1,GETDATE()),0)
AND ac.presence_status_cd = 'Present'

```


