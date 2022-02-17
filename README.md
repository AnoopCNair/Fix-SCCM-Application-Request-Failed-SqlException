# Fix-SCCM-Application-Request-Failed-SqlException

You can use the following query to fix the issue explained in the following post.

https://www.anoopcnair.com/sccm-application-request-failed-deadlock/

 
Name	timestamp_utc	message	session_id	object_type	statement	object_id
sp_statement_starting	2021-12-11 11:30:22.1510406	NULL	94	PROC	IF NOT EXISTS (SELECT 1 FROM AlertEvents WHERE EventInstanceID = @RequestGUID AND IsClosed = 1)	270057510
sp_statement_completed	2021-12-11 11:30:22.1511179	NULL	94	PROC	IF NOT EXISTS (SELECT 1 FROM AlertEvents WHERE EventInstanceID = @RequestGUID AND IsClosed = 1)	270057510
sp_statement_starting	2021-12-11 11:30:22.1511444	NULL	94	PROC	INSERT INTO AlertEvents (AlertId, EventInstanceID, IsClosed, EventTime, EventData)                   SELECT a.AlertID, @RequestGUID, 0, GETDATE(), '' from UserTargetedAppModelSoftware us                  JOIN Alert a on a.TypeInstanceID = us.OfferID                  WHERE us.AppID = @ApplicationID and us.UserID = @UserID	270057510
error_reported	2021-12-11 11:30:22.1520299	Violation of UNIQUE KEY constraint 'AlertEvents_AK'. Cannot insert duplicate key in object 'dbo.AlertEvents'. The duplicate key value is (16777968, 1, 90B6EA29-ACA1-4120-884C-8D72E50C6FE4, Dec 11 2021  6:30AM).	94	NULL	NULL	NULL
 SQL Query
 
USE [CM_HTMD] USE YOUR DB NAME
GO
/****** Object: StoredProcedure [dbo].[usp_CreateUserApplicationRequest] Script Date: 12/11/2021 7:08:20 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
--
-- Name : usp_CreateUserApplicationRequest
-- Version : 5.0.9058.1014
-- Definition : ServicePortalSqlObjs
-- Scope : PRIMARY
-- Object : P
-- Dependencies : <Detect>
-- Description : Creates a new request for the user and the input machine
-- Sproc return value is 0 for success, <0 for error
 
ALTER PROCEDURE [dbo].[usp_CreateUserApplicationRequest]
@ClientGuid NVARCHAR(255),
@User NVARCHAR(256),
@AadUser uniqueidentifier = NULL,
@AadTenantId uniqueidentifier = NULL,
@ApplicationID NVARCHAR(300),
@Comments NVARCHAR(2000),
@RequestState int output,
@RequestGUID nvarchar(256) = null, --Maps to UserApplicationRequests.RequestGuid column
@CurrentDate DATETIME = null,
@ClientSource INT = 0 --Default to unknown client source when not specified
AS
BEGIN
…
 
if @EmailApprovalFeatureOn = 1
BEGIN
IF NOT EXISTS (SELECT 1 FROM AlertEvents WHERE EventInstanceID = @RequestGUID AND IsClosed = 1)
BEGIN
       INSERT INTO AlertEvents (AlertId, EventInstanceID, IsClosed, EventTime, EventData)
       SELECT a.AlertID, @RequestGUID, 0, GETDATE(), '' from UserTargetedAppModelSoftware us
       JOIN Alert a on a.TypeInstanceID = us.OfferID
       WHERE us.AppID = @ApplicationID and us.UserID = @UserID
END
ELSE
BEGIN
       UPDATE AlertEvents SET IsClosed = 0 WHERE EventInstanceID = @RequestGUID
END
SET @ReturnValue = 0
END
RETURN @ReturnValue
END
END
 
The INSERT statement highlighted above tries to INSERT data based on the results of a SELECT statement. To confirm if the SELECT was returning any data (which was the expected behaviour) we tried to run the SELECT statement directly on SSMS as indicated below (the [?] was replaced the by a proper UserID value (obtained from the User_DISC table) but there weren’t returned results
 
SELECT 0, GETDATE(), '' from UserTargetedAppModelSoftware us 
       WHERE us.AppID = N'ScopeId_BBBB1F95-B838-4F26-A1FA-680AE77EDD68/Application_fe0ed63a-a8d0-4e63-830c-ba261f9ef24e' and us.UserID = [?] 
 
