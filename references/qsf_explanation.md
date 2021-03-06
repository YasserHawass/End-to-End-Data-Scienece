# Quickstart Guide to undertsanding the Qualtrics Survey File

This information is likely to quickly become outdated when Qualtrics next changes the formatting of the QSF file. This guide was started February 2017. I hope that it is a useful introduction to understanding the contents of the QSF file that one can download from Qualtrics. 

This document includes:
- [An introduction to JSON](https://gist.github.com/ctesta01/d4255959dace01431fb90618d1e8c241#introduction-to-json)
- [The Anatomy of a Qualtrics Survey File (QSF)](https://gist.github.com/ctesta01/d4255959dace01431fb90618d1e8c241#anatomy-of-a-qualtrics-survey-file)
- [A Breakdown of Question Structure](https://gist.github.com/ctesta01/d4255959dace01431fb90618d1e8c241#breakdown-of-question-structure)
- [A Guide to Recognizing Question Types](https://gist.github.com/ctesta01/d4255959dace01431fb90618d1e8c241#recognizing-question-types)

### Introduction to JSON

Qualtrics surveys have two components which can be exported -- their survey file and their response data. The survey file contains information about the survey's details, its contents, and all of the information necessary to reconstruct the survey. The ".qsf" filetype stands for "Qualtrics Survey File" but the contents of the file is JSON -- JavaScript Object Notation. 

[JSON Introduction on w3m](https://www.w3schools.com/js/js_json_intro.asp)

The point of JSON is that it is easily processed by machines -- and less so for humans. If you would like take a look at a QSF or JSON file, I recommend uploading it to a JSON viewer or using a "pretty printer" to render it more readable. The QSF file rarely contains confidential information, but please be careful not to upload sensitive information to an online JSON viewer.

Raw QSF Data:
<center>
<img width=500px src="http://i.imgur.com/Iz9dWqH.png" />
</center>

JSON using a [JSON Viewer](http://jsonviewer.stack.hu/)
<center>
<img width=500px src="http://i.imgur.com/k1yzXNa.png" />
</center>

Some packages to handle JSON data:
- [tidyjson in R by Hadley Wickham](https://cran.r-project.org/web/packages/tidyjson/vignettes/introduction-to-tidyjson.html)
- [json and jsonlite in R](https://cran.r-project.org/web/packages/jsonlite/vignettes/json-aaquickstart.html)
- [json in python](https://docs.python.org/3/library/json.html)

### Anatomy of a Qualtrics Survey File

A QSF contains two top level objects: <b>SurveyEntry</b> and <b>SurveyElements</b>. 

<b>SurveyEntry</b> describes many meta-details that describe the QSF. It contains the name of the survey, the creation date, start date, end date, many internal ID values that Qualtrics associates with a survey, such as the SurveyID, SurveyOwnerID, and more. 

    "SurveyEntry": {
      "SurveyID": "SV_4I2LK0XhHYRhhrL",
      "SurveyName": "Sample Survey",
      "SurveyDescription": null,
      "SurveyOwnerID": "UR_8kzO7vEaBPWnAB7",
      "SurveyBrandID": "tufts",
      "DivisionID": null,
      "SurveyLanguage": "EN",
      "SurveyActiveResponseSet": "RS_bjfzpALWe8AMZOR",
      "SurveyStatus": "Inactive",
      "SurveyStartDate": "0000-00-00 00:00:00",
      "SurveyExpirationDate": "0000-00-00 00:00:00",
      "SurveyCreationDate": "2017-02-10 08:53:57",
      "CreatorID": "UR_8kzO7vEaBPWnAB7",
      "LastModified": "2017-02-13 14:23:41",
      "LastAccessed": "0000-00-00 00:00:00",
      "LastActivated": "0000-00-00 00:00:00",
      "Deleted": null
    }
    
```R
# When you run get_setup() one of the objects returned to the global environment is survey. 
# To access SurveyEntry, access the list element survey[['SurveyEntry']].
> survey[['SurveyEntry']]

$SurveyID
[1] "SV_eFjHq9W5eLCeJI9"

$SurveyName
[1] "Long & Exhaustive Sample Survey"
...
```
    
The other top level entry, <b>SurveyElements</b>, is a numbered list that contains the major components of a survey. These roughly break down into the following different kinds of components:

  - Survey Blocks
  - Survey Flow
  - Notes
  - Survey Options
  - Scoring
  - Survey Statistics
  - Question Count
  - Survey Questions
  - Response Set
  
```R
# Similarly, to access these, access the list element survey[['SurveyElements']]. 
# However, I would caution against doing this directly, since it will print
# the whole survey to the command line. One can use functions like head and str
# to read the data a little more clearly. 
> head(survey[['SurveyElements']], n=1)

[[1]]
[[1]]$SurveyID
[1] "SV_eFjHq9W5eLCeJI9"

[[1]]$Element
[1] "BL"

[[1]]$PrimaryAttribute
[1] "Survey Blocks"
...

> str(survey[['SurveyElements']], max.level = 2)

List of 55
 $ :List of 6
  ..$ SurveyID          : chr "SV_eFjHq9W5eLCeJI9"
  ..$ Element           : chr "BL"
  ..$ PrimaryAttribute  : chr "Survey Blocks"
  ..$ SecondaryAttribute: NULL
  ..$ TertiaryAttribute : NULL
  ..$ Payload           :List of 8
  ...
```

This is what the SurveyElements can look like uploading the QSF to a JSON viewer. 

<table style="width:100%">
  <tr>
    <th><img width=200px src="http://i.imgur.com/pyaAXBZ.png"></th>
    <th><img width=200px src="http://i.imgur.com/5Roqu7h.png"></th>
  </tr>
</table> 

Note that some of these components will only ever appear once in a given survey's list of SurveyElements, while others may appear many times. I will only describe some of these which are of the most importance to the QualtricsTools project. 

Here is what I know about each of these components:

<table style="width:100%">
  <tr>
    <td>
    <img width=200px src="http://i.imgur.com/PMB2Mdd.png">
    </td>
    <td><b>Survey Blocks</b> only appears once in the list of SurveyElements as a block with PrimaryAttribute: "SurveyBlocks" and Element: "BL". It contains a list of blocks and for each it includes the block name, block ID, and a list of the export tags of the questions contained in that block. 
    </td>
  </tr>
  <tr>
    <td>
    <img width=200px src="http://i.imgur.com/xxJ5E8b.png">
    </td>
    <td><b>Survey Flow</b> always appears once in the QSF. Its payload contains a list of blocks ordered according to the flow of the survey.
    </td>
  </tr>
  <tr>
    <td><img width=200px src="http://i.imgur.com/acnD8MP.png"></td>
    <td>
    A <b>Notes</b> Survey Element contains the notes for a particular question. In a note's Payload is a "ParentID" which contains the Data Export Tag of the question the notes correspond to, and also in Payload under "Notes" there is an element for each note on the question and its "Message" containing the contents of the note. 
    </td>
  </tr>
  <tr>
    <td><img width=200px src="http://i.imgur.com/kzIGHV3.png"></td>
    <td>
    <b>Survey Options</b> contain useful information that detail what options survey respondents have when taking the survey,
    such as whether or not they can use the browser's back button, whether or not a previous question button appears, etc.
    </td>
  </tr>
  <tr>
    <td><img width=200px src="http://i.imgur.com/03jhZt2.png"></td>
    <td>
    I've never used the <b>Scoring</b>.
    </td>
  </tr>
  <tr>
    <td><img width=200px src="http://i.imgur.com/AyMNCu2.png"></td>
    <td>
    I've never used the <b>Survey Statistics</b>.
    </td>
  </tr>
  <tr>
    <td><img width=200px src="http://i.imgur.com/YEe331K.png"></td>
    <td>
    The <b>Question Count</b> is the number of questions in the QSF.
    </td>
  </tr>
  <tr>
    <td><img width=200px src="http://i.imgur.com/rldvG7x.png"></td>
    <td>
    The <b>Survey Questions</b> are stored each individually as their own Survey Element. Each question contains a PrimaryElement which is its Question ID, a SecondaryElement which is the first 99 characters of the Question Text, and of course it contains a Payload with the details of the question. See below for further breakdown of the QSF's Survey Question structure.
    </td>
  </tr>
  <tr>
    <td><img width=200px src="http://i.imgur.com/lb3acNd.png"></td>
    <td>
    The <b>Response Set</b> contains the ID of the set of respondents intended to receive or who have received the survey.
    </td>
  </tr>
</table> 

### Breakdown of Question Structure

In the QSF under the SurveyElements there will be one element for each question. Questions have many different components, the most important of which are the following:

  - SurveyID
  - Element
  - PrimaryAttribute
  - SecondaryAttribute
  - TertiaryAttribute
  - Payload
    - QuestionText
    - DataExportTag
    - QuestionType
    - Selector
    - SubSelector
    - Choices
    - RecodeValues
    - ChoiceDataExportTags
    - ChoiceOrder
    - Answers
    - AnswerOrder
    - QuestionID

If we're in R and have imported a survey, we can get a question and take a look at where these properties live within it. 

If you import a QSF and CSV by using the `get_setup` function in QualtricsTools, then the function will construct the `questions` list which you can access in the global scope. Questions are retrieved from this list via their index, and 
there are some functions in QualtricsTools to help you find a given question:

```R
> questions[[1]]
$SurveyID
[1] "SV_4ILRhqlGA79u2Md"
$Element
[1] "SQ"
$PrimaryAttribute
[1] "QID3"
...

> find_question_index(questions, "q3_volunteer")
[1] 6

> find_question_index_by_qid(questions, "QID5")
[1] 6

> questions[[6]][['Payload']][['DataExportTag']]
[1] "q3_volunteer"

> questions[[6]][['PrimaryAttribute']]
[1] "QID5"
```

Now I will explain many of the important components of a question as stored in the QSF data:

<table style="width:100%">
  <tr>
    <td>
    SurveyID
    </td>
    <td>
    The SurveyID merely indicates the survey to which a question belongs.
    </td>
  </tr>
  <tr>
    <td>
    Element
    </td>
    <td>
    A question starts out as an element of the SurveyEntries list. The Element tag simply denotes what kind
    of SurveyEntry this element is, and so for a question the Element item will always be "SQ" to denote
    survey question. That is for any question in questions, questions[[i]][['Element']] == SQ.
    </td>
  </tr>
  <tr>
    <td>
    PrimaryAttribute
    </td>
    <td>
    The PrimaryAttribute is the same as the QuestionID for questions. It is always "QID" followed by some numbers.
    The numbers usually go in the order of the creation of questions as the survey was made. These QIDs are distinct
    from the DataExportTags which users can customize in the Qualtrics interface. QIDs cannot be changed. 
    </td>
  </tr>
  <tr>
    <td>
    SecondaryAttribute
    </td>
    <td>
    This list item contains up to a certain number of characters of the question text. If the question is particularly long, the SecondaryAttribute does not always contain the full question text. For that, look to the question[['Payload']][['QuestionText']] element.
    </td>
  </tr>
  <tr>
    <td>
    TertiaryAttribute
    </td>
    <td>
    I have never seen a question that had a non-NULL TertiaryAttribute.
    </td>
  </tr>
<tr>
    <td>
    Payload
    </td>
    <td>
    The Payload contains the remaining elements which describe a given question.
    </td>
  </tr>

    <tr>
    <td>
    QuestionText
    </td>
    <td>
    The QuestionText element contains the question's text.
    </td>
  </tr>

  <tr>
    <td>
    DataExportTag
    </td>
    <td>
    The DataExportTag is the tag for a given question which is customizable in the interface <a href='http://i.imgur.com/3X9SJu6.png'>like so</a>. 
    </td>
  </tr>
    <tr>
    <td>
    QuestionType
    </td>
    <td>
    The QuestionType contains a short code which describes which of the basic categories of questions
    this question falls into. These can be "MC", "TE", "DB", "Matrix", "SBS", "DD", and others. It is
    my guess that these stand for respectively: "Multiple Choice", "Text Entry", "Descriptive Box", 
    "Matrix", "Side-by-Side", "Drop-Down", etc.
    </td>
  </tr>
  <tr>
    <td>
    Selector
    </td>
    <td>
    The Selector describes what interface the respondent uses to make their response to the question. 
    These range from things like "MAVR", "TE", "Likert", "SAVR", "TB", and more. These abbreviations
    stand for "Multiple Answer Vertical", "Text Entry", "Likert", "Single Answer Vertical", "Text-Box",
    etc.
    "
    </td>
  </tr>
  <tr>
    <td>
    SubSelector
    </td>
    <td>
    Only some questions have SubSelector elements within their Payload. The SubSelector may 
    contain information like "SingleAnswer" or "MultipleAnswer" for a Matrix Likert question,
    and for other questions it contains strings like "TX" and "Long".
    </td>
  </tr>
  <tr>
    <td>
    Choices
    </td>
    <td>
    These are the choices which were presented to a respondent. They are indexed by internal labeling,
    and if you have renamed them in the Qualtrics interface, then their naming convention 
    will show up in the RecodeValues. If the question's choices are ordered in a particular way, 
    then this will show up in ChoiceOrder.
    </td>
  </tr>
  <tr>
    <td>
    RecodeValues
    </td>
    <td>
    The RecodeValues of a question are how the names of the possible choices were customized. These
    are indexed in the same order as the Choices list, and their contents is the customized label for
    the ith choice, i.e. question[['Choices']][[i]] is labeled question[['RecodeValues']][[i]] if
    RecodeValues are being used. If a question's choices were not recoded, this list element does not appear.
    
    See [Recode Values menu option](http://i.imgur.com/80iKEqS.png) and [recode values interface](http://i.imgur.com/2uczrrW.png)
    </td>
  </tr>
  <tr>
    <td>
    ChoiceDataExportTags
    </td>
    <td>
    In the context of a matrix question, the question's Choices are considered to be the horizontal elements of
    the matrix. These can not only be recoded in the interface, but can also be given alphanumeric names. This
    is a list which is indexed with the same indices as the Choices list which contains these prescribed 
    alphanumeric names.
    
    The ChoiceDataExportTags are listed in the Qualtrics interface as <a href='http://i.imgur.com/XxYDWFn.png'>Question Export Tags</a>
    </td>
  </tr>
  <tr>
    <td>
    ChoiceOrder
    </td>
    <td>
    This ordering dictates in which order the choices are listed on the screen. 
    </td>
  </tr>
  <tr>
    <td>
    Answers
    </td>
    <td>
    For Matrix questions, the vertical options are denoted as "Answers" in the question's structure. These answers can also be renamed, and if they are this information is included under RecodeValues.
    </td>
  </tr>
  <tr>
    <td>
    AnswerOrder
    </td>
    <td>
    The AnswerOrder list describes the order in which the vertical options of a matrix question are listed. 
    </td>
  </tr>
  <tr>
    <td>
    QuestionID
    </td>
    <td>
    The QuestionID is the ID that is automatically assigned to a question. This is not editable, unlike the DataExportTag. 
    </td>
  </tr>
</table> 

### Recognizing Question Types

To determine how the question's results are supposed to be determined, it's very important that we are able to quickly 
determine what kind of question a given question is. To do so, I have written the functions contained in the <a href='https://github.com/ctesta01/QualtricsTools/blob/master/R/question_type_checking.R'>question_type_checking.R file</a>


<table style="width:100%">
  <tr>
    <td>
    <a href="https://gist.github.com/anonymous/8f1c111ab2c02681ac150ea6d3f5f180">MCSA</a>
    </td>
    <td>
    Multiple Choice Single Answer questions are characterized by having question[['Payload']] == "MC",
    and `question[['Payload']][['Selector']]` is one of "SAVR", "SAHR", "SACOL", "DL", or "SB".
    </td>
  </tr>
  <tr>
    <td>
    <a href="https://gist.github.com/anonymous/a7a2b11a6006d5cdb9c047ee7a69548a">MCMA</a>
    </td>
    <td>
    Multiple Choice Multiple Answer questions are characterized by having question[['Payload']] == "MC",
    and `question[['Payload']][['Selector']]` is one of "MAVR", "MAHR", "MACOL", or "MSB".
    </td>
  </tr>
  <tr>
    <td><a href="https://gist.github.com/anonymous/f68c90fff8cfcc84268dc014a1211078">TE</a>
    <td>
    Text Entry questions have question[['Payload']][['QuestionType']] == "TE".
    </td>
  </tr>
  <tr>
    <td><a href="https://gist.github.com/anonymous/0d32b330b7f7ea691314996916861400">Likert SA</a>
    </td>
    <td>
    A Matrix Single Answer question has question[['Payload']][['QuestionType']] == "Matrix"
    and has SubSelector as one of "DL" or "SingleAnswer".
    A Matrix question has both Choices and Answers, and may potentially have ChoiceDataExportTags
    and RecodeValues to label these. 

    </td>
  </tr>
  <tr>
    <td><a href="https://gist.github.com/anonymous/28dc79da2f0338b5b085dff3c5b52b8d">Likert MA</a>
    </td>
    <td>
    A Matrix Single Answer question has question[['Payload']][['QuestionType']] == "Matrix"
    and has SubSelector as "MultipleAnswer"
    A Matrix question has both Choices and Answers, and may potentially have ChoiceDataExportTags
    and RecodeValues to label these. 
    </td>
  </tr>
  <tr>
    <td><a href="https://gist.github.com/anonymous/f8b867be1e05c8df8a6ccd8a8f1ffbf9">SBS</a>
    </td>
    <td>
    Side By Side questions have question[['Payload']][['QuestionType']] == "SBS". 
    They are automatically broken down by the QualtricsTools application into smaller questions, 
    and reinserted as independent questions. The contents of their contained questions 
    is kept within the question's [['Payload']][['AdditionalQuestions']] list. 
    </td>
  </tr>
</table> 

