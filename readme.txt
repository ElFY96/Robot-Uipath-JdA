We create all our workflows into 'Junta de Andalucia' Folder, except 'saveFile.xaml' we create it in high leel with 'main.xaml,process.xaml...', 
because we can use it in any robot, not only in this robot.

Initialization
**************

  -Initialize All Settings
   -----------------------
    # In order to change our output data(contains robot result) folder and input data(contain config.xlsx and Filters.xlsx) data folders position from the default position 
      to desktop,we should change in_ConfigFile arguments value to C:\Users\Yassin El Fadili\Desktop\input data, but as we can notice, our robot will not work in
      other machines because of "C:\Users\Yassin El Fadili", it is a dynamic path. I suggest as a solution to set this dynamic path using 
      'Environment.ExpandEnvironmentVariables("%USERPROFILE%\Desktop")', then we should concatinate the dynamic part with the fixed part to get full path. And we shold
      initialize in_ConfigFile in for each loop with value "\input data\Config.xlsx" to avoid concatination problems inside loop, if we did not we will get something
      like that "C:\Users\Yassin El Fadili\input data\Config.xlsx\input data\Config.xlsx" and our robot will not recognize this path, so an error will occur.
    # We will use this dynamic path as output argument because we will need it in others operations.
    # We use kill activety to make sure IE browser close. 

  -OpenJuntaDeAndalucia.xaml
   -------------------------
  -Retry Scope activity to make our robot retry opening junta de Andalucia web site if it get failed to open.( we set number of retries to 3 )
    # Inside the Retry Scope activity we use :
        * Element exists activity to check if IE is open then close it ( that will happened just if browser failed to open, then open will close before retry open it).
	* We will use open browser activity and open an empty tab, then maximaze it and use navigate to activity to open junta de Andalucia web site ( we suggest this 
         solution because when we send url directly to open browser activety, sometimes 2 IE windows opens in the same time).
    # Once Our web site opened successfully, robot will create a folder inside output data take as a name today date. if an error occur and robot did not pass 
      to Get Transaction Data this folder will be delete. 

Get Transaction Data
********************
  -First, we should make sure that we change 'TransactionItem' variable type from Queueitem to String (because we are not going to work with queue in this robot).
   Change if condition to 'in_TransactionNumber <=1' because we want to execute just one transactionItem.(If we want to execute more than 1, then we should change 
   'TransactionItem' to DataRow).
  -When condition is not met we will give nothing value to 'out_TransactionItem', that will make robot know that there is no more transactions, and go to end process.
  - Change 'New Transaction' condition to 'in_TransactionNumber <=1', when the result of condition is True, our robot will move to 'Process Transaction'.

Process Transaction
*******************

  -getFilters.xaml
   ---------------
    # We will use Excel Application Scope with 'in_dynamicPath+in_filtresFile' (in_filtresFile=in_Config("Filters_ExcelFile").ToString) as a path,
      to get filters.xlsx file. Inside , the scope we will use read cell activity, to red data from our Filters.xlsx file.
    # We will use split method with ',' to get array of Strings , then put each array row in a new variable, at the last, set those new variables as output data in
      order to use its as a filters.

  -fillSearchFilters.xaml
   ----------------------
    # Set inputs variables as a filters, and click buscar button, when result search appear,let results number and remove the rest of text, after that, convert String
      value to integer and set result number as an output.

  -copyContentToExcelFile.xaml
   ---------------------------  
    # We will use dataScraping to get extact data table (contains links), and we should use the input result number as a MaxNumberOfResults(if we did not the robot will 
      enter in an infinity loop of data extraction).And we will give to fileName variable, result of 'in_folderName + "\Resultados.xlsx"'.After that we will initialize
      an index with 1 as initial value, inside an excel scope with fileName as workbook path, for each row from the extracted data table :
            * we will increment index (that mean the index = 2 and greater).
            * write columns title usig write cell.
            * write culumns contents, using write cell and index as number of cellul. To write links contents,
              we use 'String.Format("=HYPERLINK(""{0}"",""{1}"")",row(X+1),row(X))' with row(X) is content name, and row(X+1) is content link.
    # When Resultados.xlsx created, we will create a sub-folder which is at the same level of Resultados.xlsx, in order to save PDF files into it. And we will set this
      sub-folder as an output variable to use it later.

   --------------------------------------------------------------------------------------------------------------------
   -inside for each row in extract data table (we get this data table as output from 'copyContentToExcelFile.xaml')
   --------------------------------------------------------------------------------------------------------------------
		* accessRowDetails.xaml
		* goToDocumentaciónComplementaria.xaml
		* downloadDocuments.xaml
   --------------------------------------------------------------------------------------------------------------------
     
   -accessInformationsRow.xaml
    --------------------------
     #We will open a new tab by sending ctrl+t key, then attach the new tab and navigate to link of specific link (it is in_Link input argument,
      whih equal row(1).ToString). row(1).ToString is a links. column. 
      
   -goToDocumentaciónComplementaria.xaml
    ------------------------------------
     #After open link wait 'Documentación complementaria' to appear, then click on 'Mostrar información'.Notice that this 'Mostrar información' button havn't its own
      features that allow to record it so we use anchor which is 'Mostrar información' to make robot recognize it.
      
   -downloadDocuments.xaml
    ------------------------------------
     #We will extract all pdf names which exists inside 'Documentación complementaria' in a data table, then for each row in this data table, click on file name using
      'file(0).ToString' as a dynamic variable for attribute 'aaname' in the click selector.
     #When save alert appear click on dropdown, then on save as  and write 'in_savePath+fileName' as file path and send enter key.































    
        
