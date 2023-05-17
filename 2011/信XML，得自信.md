# 信XML，得自信
>date: 2011-01-19T08:49:48+08:00


XML可能是计算有史以来最NB的发明了，以至于我们以没有XML的程序是难登大堂的程序，不用XML，你都不好意思当程序员。于是，我们看到了[很多很雷人的用法](/2010/%E4%BF%A1XML%EF%BC%8C%E5%BE%97%E6%B0%B8%E7%94%9F%EF%BC%81.md)（《信XML，得永生》），当然一些朋友当时并没有看懂，不过我不怪大家，因为我们依然深信使用XML可以让你有强大的Zhuangbility，于是我们有下面这两种相当Geiliable的用法。


#### 一、XML中的XML


这个例子是某公司的一个SOAP实现——我们的Webservice需要返回一个XML字符串，这怎么办呢？其实很容易，因为——XML是无所不能的，那怕是封装自己。



```
<!-- ED: soap envelope omitted for readability -->
<string xmlns="urn:Initech.Global.Services">
  &lt;CompanyGetConnector&gt;
    &lt;xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"&gt;
      &lt;xs:element name="InitechGetConnector"&gt;
        &lt;xs:complexType&gt;
          &lt;xs:choice maxOccurs="unbounded"&gt;
            &lt;xs:element name="employees"&gt;
              &lt;xs:complexType&gt;
                &lt;xs:sequence&gt;
                  &lt;xs:element name="EmployerName" type="xs:string" minOccurs="0"/&gt;
                  &lt;xs:element name="Employee" type="xs:string" minOccurs="0"/&gt;
                  &lt;xs:element name="Firstname" type="xs:string" minOccurs="0"/&gt;
                  &lt;xs:element name="Prefix" type="xs:string" minOccurs="0"/&gt;
                  &lt;xs:element name="Lastname" type="xs:string" minOccurs="0"/&gt;
                  &lt;xs:element name="Org._unit" type="xs:string" minOccurs="0"/&gt;
                  &lt;xs:element name="Function" type="xs:string" minOccurs="0"/&gt;
                  &lt;xs:element name="E-mail_work" type="xs:string" minOccurs="0"/&gt;
                  &lt;xs:element name="Telephone_work" type="xs:string" minOccurs="0"/&gt;
                  &lt;xs:element name="Mobile_work" type="xs:string" minOccurs="0"/&gt;
                  &lt;xs:element name="Birthdate" type="xs:date" minOccurs="0"/&gt;
                  &lt;xs:element name="Hired_since__irt._yearsemployed_" type="xs:date" minOccurs="0"/&gt;
                  &lt;xs:element name="Image" type="xs:base64Binary" minOccurs="0"/&gt;
                &lt;/xs:sequence&gt;
              &lt;/xs:complexType&gt;
            &lt;/xs:element&gt;
          &lt;/xs:choice&gt;
        &lt;/xs:complexType&gt;
      &lt;/xs:element&gt;
    &lt;/xs:schema&gt;

    &lt;employees&gt;
      &lt;EmployerName&gt;
        My Client
      &lt;/EmployerName&gt;
      &lt;Employee&gt;
        100001
      &lt;/Employee&gt;
    &lt;/employees&gt;
  &lt;/CompanyGetConnector&gt;
</string>

```


#### 二、一切皆为配置


没有hard code这是一个优秀程序员在入门时就要学习的，对于Hard Coder的东西最好写在配置文件中，这样修改这些参数就不需要修改代码而需要重新编译了。自从有了XML之后，我们的配置文件就不在使用像ini文件或是Unix下在conf文件那样的易读，我们认为，使用XML作为配置文件的格式是大势所趋，而且，我们要让我们的代码尽量的可以高度的配置，于是我们出现了下面的代码——这是一个强大的尝试，其标志着，我们完全可以以不久的未来用XML来编写一切语言的代码。


注：下面的代码最强大的应该是XML中的那个SQL。



```
<add key="sqlSource" value="
    SELECT TOP REPLACE_NUMBER_OF_ROWS_TO_RETRIEVE
           History.handle AS ID_FAX_LOG,
           CASE isnumeric(SUBSTRING (Notes_Doc.Text ,1,8))
              WHEN 1 then SUBSTRING (Notes_Doc.Text ,1,8)
              ELSE NULL END AS ID_STAGE,
           DocumentUsers.UserName AS NM_DOCUMENTUSER_USERNAME,
           DocumentUsers.UserID AS TXT_DOCUMENTUSER_USERID,
           DocumentUserGroups.GroupID AS TXT_DOCUMENTUSERGROUP_GROUPID,
           Documents.UniqueID AS TXT_DOCUMENTS_UNIQUE_ID,
           History.TRDateTime AS DT_HISTORY_TRANSACTION_DATE,
           CASE COALESCE(HistoryPrint.handle,0)
              WHEN 0 THEN
                 CASE COALESCE(HistoryGeneric.handle,0)
                    WHEN 0 THEN
                       CASE COALESCE(HistoryTRX.handle,0)
                          WHEN 0 THEN '??'
                          ELSE
                             CASE (Documents.Flags & 0x10)
                                WHEN 0 THEN 'Send'
                                ELSE 'Recieve'
                                END
                          END
                    ELSE CAST(HistoryGeneric_Short.Data AS varchar(32))
                    END
              ELSE 'Print'
              END AS TXT_TRANSACTION_TYPE,

           CASE COALESCE(HistoryPrint.handle,0)
              WHEN 0 THEN
                 CASE COALESCE(HistoryGeneric.handle,0)
                    WHEN 0 THEN
                       CASE COALESCE(HistoryTRX.handle,0)
                          WHEN 0 THEN '??'
                          ELSE
                             CASE Documents_Term.TermStatStr
                                WHEN 'Success' THEN 'Success'
                                ELSE 'Fail'
                                END
                          END
                    ELSE
                       CASE HistoryGeneric.ErrCode
                          WHEN 0 THEN 'Success'
                       ELSE 'Fail'
                       END
                    END
              ELSE
                 CASE SUBSTRING(HistoryPrint.Msg,1,7)
                    WHEN 'Success' THEN 'Success'
                    ELSE 'Fail'
                    END
              END AS TXT_TRANSACTION_STATUS,

           CASE COALESCE(HistoryPrint.handle,0)
              WHEN 0 THEN
                 CASE COALESCE(HistoryGeneric.handle,0)
                    WHEN 0 THEN
                       CASE COALESCE(HistoryTRX.handle,0)
                          WHEN 0 THEN '??'
                          ELSE COALESCE(HistoryTRX_Term.TermStatStr,CONVERT(varchar,Documents.TermStat))
                          END
                    ELSE REPLACE(REPLACE(CAST(HistoryGeneric_Detail.Data AS varchar(192)) ,'\t',''), '~u', HistoryGeneric.UserID )
                    END
              ELSE HistoryPrint.Msg
              END AS TXT_TRANSACTION_MESSAGE,

           CASE COALESCE(HistoryPrint.handle,0)
              WHEN 0 THEN
                 CASE COALESCE(HistoryGeneric.handle,0)
                    WHEN 0 THEN
                       CASE COALESCE(HistoryTRX.handle,0)
                          WHEN 0 THEN Documents.ElapsedSendTime
                          ELSE
                             CASE COALESCE(HistoryTRX.handle,0)
                                WHEN 0 THEN Documents.ElapsedSendTime
                                ELSE HistoryTRX.ElapsedTime
                                END
                          END
                    ELSE NULL
                    END
              ELSE HistoryPrint.TimeToPrint
              END AS NBR_TRANSACTION_ELAPSEDTIME,

           CASE COALESCE(HistoryGeneric.handle,0)
              WHEN 0 THEN
                 CASE substring(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE
                               (REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(
                                  Documents.Destination,' ',''),')',''),'(',''),
                                  '-',''),'/',''),'.',''),'*',''),',',''),';',''),
                                  '\',''),'-',''),1,1)
                    WHEN '1' THEN substring(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(
                                            REPLACE(REPLACE(REPLACE( REPLACE(REPLACE(
                                            REPLACE(Documents.Destination,' ',''),')',
                                            ''),'(',''),'-',''),'/',''),'.',''),'*',''),
                                            ',',''),';',''),'\',''),'-',''), 2, len(
                                            REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(
                                            REPLACE(REPLACE(REPLACE( REPLACE(REPLACE(
                                            REPLACE(Documents.Destination,' ',''),')',
                                            ''),'(',''),'-',''),'/',''),'.',''),'*','')
                                            ,',',''),';',''),'\',''),'-','')) )
                    ELSE REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(
                         REPLACE(REPLACE(REPLACE(Documents.Destination,' ',''),'-',''),')',
                         ''),'(',''),'/',''),'.',''),'*',''),',',''),';',''),'\',''),'-','')
                    END
              ELSE HistoryGeneric.UserID
              END AS TXT_TRANSACTION_DESTINATION,

           CASE (Documents.Flags & 0x8)
              WHEN 0 THEN 'N'
              ELSE 'Y' END AS NBR_DOCUMENTS_DELETED,

           CASE (Documents.Flags & 0x4)
              WHEN 0 THEN 'N'
              ELSE 'Y' END AS NBR_DOCUMENTS_VIEWED,

           /* Fax Destination */
           Documents.ToName AS TXT_DOCUMENTS_TO_NAME,
           Documents.ToContactNum AS TXT_DOCUMENTS_TO_CONTACT_NUM,
           Documents.ToCompany AS TXT_DOCUMENTS_TO_COMPANY,
           Documents.ToCityState AS TXT_DOCUMENTS_TO_CITY_STATE,
           Documents.FaxDIDNum AS TXT_DOCUMENTS_FAX_DID_NUM,
           Documents.FromPhoneNum AS TXT_DOCUMENTS_FROM_PHONE_NUM,
           Documents.GeneralFaxNum AS TXT_DOCUMENTS_GENERAL_FAX_NUM,
           HistoryPrint.NetPrintID AS TXT_HISTORYPRINT_NETPRINTID,

           /* Number of pages */
           DocFiles.NumPages AS NBR_DOCFILES_TOTAL_PAGE_COUNT,
           HistoryTRX.GoodPageCount AS NBR_HISTORYTRX_GOOD_PAGE_COUNT,
           HistoryTRX.BadPageCount AS NBR_HISTORYTRX_BAD_PAGE_COUNT,
           HistoryPrint.PagesPrinted AS NBR_HISTORYPRINT_PAGESPRINTED,
           HistoryPrint.CopiesPrinted AS NBR_HISTORYPRINT_COPIESPRINTED,
           /* location of fax image */ DTConfigurations.ServerName AS TXT_DOCFILES_SERVER_NAME,
           DTConfigurations.ImageDir AS TXT_DOCFILES_IMAGE_DIR,
           DocFiles.BodyFilename AS TXT_DOCFILES_BODY_FILENAME,
           Documents.FCSFile AS TXT_DOCFILES_FCS_FILE,
           REPLACE( DTConfigurations.ImageDir, 'D:\Data', '\\'+ServerName )
              + '\'+DocFiles.BodyFilename+'*' AS TXT_DOCFILES_PATH_BODY_NAME,
           REPLACE( DTConfigurations.ImageDir, 'D:\Data', '\\'+ServerName )
              + '\'+Documents.FCSFile+'*' AS TXT_DOCUMENTS_PATH_FCSFILE,
           Notes_Doc.Text AS TXT_NOTES_DOC_TEXT,
           Notes_CCList.Text AS TXT_NOTES_CCLIST_TEXT,
           DocumentUsers.RouteInfo AS TXT_DOCUMENTUSER_ROUTEINFO,
           DocumentUsers.RouteType AS NBR_DOCUMENTUSER_ROUTETYPE,
           DocumentUsers.EmailAddr AS TXT_DOCUMENTUSER_EMAILADDR,

           /* misc Documents data */
           Documents.CreationTime AS DT_DOCUMENTS_CREATION_TIME,
           Documents.FRFlags2 AS NBR_DOCUMENTS_FRFLAGS2,
           Documents.Flags AS NBR_DOCUMENTS_FLAGS,
           Documents.ErrorCode AS NBR_DOCUMENTS_ERROR_CODE,
           Documents.TermStat AS NBR_DOCUMENTS_TERMSTAT,

           /* misc HistoryTRX data */
           HistoryTRX.RemoteID AS TXT_HISTORYTRX_REMOTE_ID,
           HistoryTRX.RemoteServer AS TXT_HISTORYTRX_REMOTE_SERVER,
           HistoryTRX.Flags AS NBR_HISTORYTRX_FLAGS,
           HistoryTRX.TermStat AS NBR_HISTORYTRX_TERMSTAT,

           /* misc HistoryTRX data */
           HistoryGeneric.ErrCode AS NBR_HISTORYGENERIC_ERRCODE,
           HistoryGeneric.GenType AS NBR_HISTORYGENERIC_GENTYPE,
           HistoryGeneric.UserID AS TXT_HISTORYGENERIC_USERID,

           /* Handles */ Documents.handle AS ID_DOCUMENTS_HANDLE,
           History.handle AS ID_HISTORY_HANDLE,
           HistoryTRX.handle AS ID_HISTORYTRX_HANDLE,
           HistoryGeneric.handle AS ID_HISTORYGENERIC_HANDLE,
           HistoryPrint.handle AS ID_HISTORYPRINT_HANDLE

    FROM Documents
            INNER JOIN Users DocumentUsers ON Documents.OwnerID = DocumentUsers.handle
            INNER JOIN History ON Documents.handle = History.Owner
            LEFT OUTER JOIN DocFiles ON Documents.DocFileDBA = DocFiles.handle
            LEFT OUTER JOIN Groups DocumentUserGroups ON DocumentUsers.GroupID = DocumentUserGroups.handle
            LEFT OUTER JOIN HistoryPrint ON HistoryPrint.handle = History.handle
            LEFT OUTER JOIN HistoryGeneric ON HistoryGeneric.handle = History.handle
            LEFT OUTER JOIN Notes Notes_Doc ON Notes_Doc.handle = Documents.NoteDBA
            LEFT OUTER JOIN Notes Notes_CCList ON Notes_CCList.handle = Documents.CCListDBA
            LEFT OUTER join DTConfigurations ON DTConfigurations.ServerGUID = Documents.ServerGUID
            LEFT OUTER JOIN Globalization HistoryGeneric_Detail ON
               HistoryGeneric_Detail.Namespace = 'RightFax.SQL.HistoryGeneric'
               AND SUBSTRING(HistoryGeneric_Detail.LocKey,5,20) = 'DetailMsg'
               AND SUBSTRING(HistoryGeneric_Detail.LocKey,1,3) = CAST(HistoryGeneric.GenType AS varchar)
               AND HistoryGeneric_Detail.IsoLanguageName = 'en-us'
            LEFT OUTER JOIN Globalization HistoryGeneric_Short ON
               HistoryGeneric_Short.Namespace = 'RightFax.SQL.HistoryGeneric'
               AND SUBSTRING(HistoryGeneric_Short.LocKey,5, 20) = 'ShortMsg'
               AND SUBSTRING(HistoryGeneric_Short.LocKey,1, 3) = CAST(HistoryGeneric.GenType AS varchar)
               AND HistoryGeneric_Short.IsoLanguageName = 'en-us'
            LEFT OUTER JOIN HistoryTRX ON HistoryTRX.handle = History.handle
            LEFT OUTER JOIN (
               SELECT distinct CONVERT(varchar,G.Data) AS TermStatStr,
                      T.StatusCode AS TermStatCode,
                      T.handle AS TermStat
                 FROM Globalization G
                         INNER JOIN TermStatToStatusCode T ON
                            ( G.LocKey = 'HistoryTRX.BTHUSTAT'
                                 + RIGHT('0000'
                                 + LTRIM(RTRIM(CONVERT(char(3),T.StatusCode))), 3)
                              AND G.IsoLanguageName = 'en-us'
                              AND G.LocKey like 'HistoryTRX.BTHUSTAT%' )
                  ) AS HistoryTRX_Term ON HistoryTRX.TermStat = HistoryTRX_Term.TermStat
            LEFT OUTER JOIN (
               SELECT distinct CONVERT(varchar,G.Data) AS TermStatStr,
                      T.StatusCode AS TermStatCode,
                      T.handle AS TermStat
                 FROM Globalization G
                         INNER JOIN TermStatToStatusCode T ON
                            ( G.LocKey = 'HistoryTRX.BTHUSTAT'
                                 + RIGHT('0000'
                                 + LTRIM(RTRIM(CONVERT(char(3),T.StatusCode))), 3)
                              AND G.IsoLanguageName = 'en-us'
                              AND G.LocKey like 'HistoryTRX.BTHUSTAT%' )
                  ) AS Documents_Term ON Documents.TermStat = Documents_Term.TermStat
    WHERE
       NOT (
          /* The outer join on the HistoryPrint, HistoryGeneric, and HistoryTRX results in
           * rows that just have null history data. One of the three must have a value. If
           * all are null, the row is a result of the outer joins and the rows have no useable data so they
           * filtered out. */
          HistoryTRX.handle IS NULL
             AND HistoryGeneric.handle IS NULL
             AND HistoryPrint.handle IS NULL )
       AND DocumentUsers.UserName IS NOT NULL

       /* THIS VALUE is inserted into a NON NULL column in the FAX_LOG table. */
       AND DocumentUsers.UserID IS NOT NULL

       /* THIS VALUE is inserted into a NON NULL column in the FAX_LOG table. */
       AND Documents.UniqueID IS NOT NULL

       /* THIS VALUE is inserted into a NON NULL column in the FAX_LOG table. */
       AND History.TRDateTime > 'REPLACE_WHERE_CLAUSE_CRITERIA'
   ORDER BY History.TRDateTime"
/> 
```

来源：[文章一](http://thedailywtf.com/Articles/All-In-The-Config.aspx)，[文章二](http://thedailywtf.com/Articles/XMLd-XML.aspx)


