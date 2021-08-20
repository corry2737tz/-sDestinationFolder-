# -sDestinationFolder-
Func _ExtractZip($sZipFile, $sFolderStructure, $sFile, $sDestinationFolder)     Local $i     Do         $i += 1         $sTempZipFolder = @TempDir &amp; "\Temporary Directory " &amp; $i &amp; " for " &amp; StringRegExpReplace($sZipFile, ".*\\", "")     Until Not FileExists($sTempZipFolder) ; this folder will be created during extraction     Local $oShell = ObjCreate("Shell.Application")     If Not IsObj($oShell) Then Return SetError(1)     Local $oDestinationFolder = $oShell.NameSpace($sDestinationFolder)     If Not IsObj($oDestinationFolder) Then Return SetError(2) ; unavailable destionation location     Local $oOriginFolder = $oShell.NameSpace($sZipFile &amp; "\" &amp; $sFolderStructure) ; FolderStructure is overstatement because of the available depth     If Not IsObj($oOriginFolder) Then Return SetError(3) ; unavailable location     ;Local $oOriginFile = $oOriginFolder.Items.Item($sFile)     Local $oOriginFile = $oOriginFolder.ParseName($sFile)     If Not IsObj($oOriginFile) Then Return SetError(4) ; no such file in ZIP file     ; copy content of origin to destination     $oDestinationFolder.CopyHere($oOriginFile, 4) ; 4 means "do not display a progress dialog box"     DirRemove($sTempZipFolder, 1) ; clean temp dir if needed     Return 1 ; All OK! EndFunc
#include "EzMySql.au3"
#include <Array.au3>

If Not _EzMySql_Startup() Then
    MsgBox(0, "Error Starting MySql", "Error: "& @error & @CR & "Error string: " & _EzMySql_ErrMsg())
    Exit
EndIf

$Pass = "Your password here"

If Not _EzMySql_Open("", "root", $Pass, "", "3306") Then
    MsgBox(0, "Error opening Database", "Error: "& @error & @CR & "Error string: " & _EzMySql_ErrMsg())
    Exit
EndIf

If Not _EzMySql_Exec("CREATE DATABASE IF NOT EXISTS EzMySqlTest") Then
    MsgBox(0, "Error opening Database", "Error: "& @error & @CR & "Error string: " & _EzMySql_ErrMsg())
    Exit
EndIf

If Not _EzMySql_SelectDB("EzMySqlTest") Then
    MsgBox(0, "Error setting Database to use", "Error: "& @error & @CR & "Error string: " & _EzMySql_ErrMsg())
    Exit
EndIf

$sMySqlStatement = "CREATE TABLE IF NOT EXISTS TestTable (" & _
                   "RowID INT NOT NULL AUTO_INCREMENT," & _
                   "Name TEXT NOT NULL ," & _
                   "Age INT NOT NULL ," & _
                   "EyeColour TEXT NOT NULL ," & _
                   "HairColour TEXT NOT NULL ," & _
                   "PRIMARY KEY (`RowID`) ," & _
                   "UNIQUE INDEX RowID_UNIQUE (`RowID` ASC) );"

If Not _EzMySql_Exec($sMySqlStatement) Then
    MsgBox(0, "Error Creating Database Table", "Error: "& @error & @CR & "Error string: " & _EzMySql_ErrMsg())
    Exit
EndIf

Local $aEyeColours[7] = ["Amber","Blue","Brown","Grey","Green","Hazel","Red"]
Local $aHairColours[6] = ["Brown","Black","Blond","Grey","Green","Pink"]

Local $sMySqlStatement = ""
For $i = 1 To 50 Step 1
    $sMySqlStatement &= "INSERT INTO TestTable (Name,Age,EyeColour,HairColour) VALUES (" & _
                        "'Person" & $i & "'," & _
                        "'" & Random(1, 100, 1) & "'," & _
                        "'" & $aEyeColours[Random(0, 6, 1)] & "'," & _
                        "'" & $aHairColours[Random(0, 5, 1)] & "');"
Next

If Not _EzMySql_Exec($sMySqlStatement) Then
    MsgBox(0, "Error inserting data to Table", "Error: "& @error & @CR & "Error string: " & _EzMySql_ErrMsg())
    Exit
EndIf

MsgBox(0, "Last AUTO_INCREMENT columnID2", _EzMySql_InsertID())

$aOk = _EzMySql_GetTable2d("SELECT Name,EyeColour FROM TestTable WHERE EyeColour = '"& $aEyeColours[Random(0, 6, 1)] & "';")
$error = @error
If Not IsArray($aOk) Then MsgBox(0, $sMySqlStatement & " error", $error)
_ArrayDisplay($aOk, "2d Array Names of certain eyecolour")

MsgBox(0, "", "Following is how to get a row at a time of a query as a 1d array")
If Not _EzMYSql_Query("SELECT * FROM TestTable WHERE HairColour = '"& $aHairColours[Random(0, 5, 1)] & "' LIMIT 5;") Then
MsgBox(0, "Query Error", "Error: "& @error & @CR & "Error string: " & _EzMySql_ErrMsg())
    Exit
EndIf

For $i = 1 To _EzMySql_Rows() Step 1
    $a1Row = _EzMySql_FetchData()
    _ArrayDisplay($a1Row, "Result: " & $i)
Next

_EzMySql_Exec("DROP TABLE TestTable")

_EzMySql_Close()
_EzMySql_ShutDown()
Exit
