# 資料庫 （PostgreSQL)

- ##### ERD：

````mermaid
graph LR
Teacher[教授] -- 1-擁有-N --> Course;
Course -- 1-擁有-N --> Problem[題目];
Problem -- 1-擁有-N --> Copy[抄襲];
Problem -- 1-擁有-N --> Judge;
Course -- 1-擁有-N --> Feedback
Course -- 1-擁有-N --> Team[隊伍]
Course[課程] -- 1-擁有-N --> Student_Course[學生_課程];
Student_Course -- N-擁有-1 --> Student[學生]
Student -- 1-擁有-N --> Feedback[回饋];
Student -- 1-擁有-N --> Judge[批改];
Course -- 1-擁有-N --> Assistant_Course[助教_課程];
Assistant_Course -- N-擁有-1 --> Assistant[助教]
````

+ Table：

1. **student**：學生

   ```json
   student_id // primary key, bigserial
   account // text, 學生學號
   password // text, 學生密碼
   name // text, 學生姓名
   student_class // text, 年度+學生系級 e.g.106資管B
   ```

2. **problem**

   ```json
   problem_id // primary key, bigserial
   course_id // foreign key, bigserial
   name // text, 題目名稱
   type // text, 題目類型(作業 || 練習題 || 討論題)
   category // text, 題目作答類型(輸入輸出 || 輸入寫檔 || 讀檔輸出 || 讀檔寫檔)
   tag // text[], 題型分類(Java || Python, 條件,迴圈)(以,為分隔)
   rate // double, 題目平均難易度,預設為0
   desc // text, 題目描述
   input_desc // text, 輸入描述
   output_desc // text, 輸出描述
   test_cases: {[ // json, 測試範本
   	"inputSample": "", // String, 輸入範本
        "outputSample": "" // String, 輸出範本
   ]}
   deadline // date, 截止日期, 格式為yyyy-mm-dd
   correct_num // integer, 此題正確(滿分)人數, 預設為0
   incorrect_num // integer, 此題錯誤(未滿分)人數, 預設為0
   correct_rate // double, 正確率, 預設為0
   best_student_account // text, 最佳代碼的學生學號(account) // 注意學生資料刪除時，須同步
   ```

3. **copy**

   ```json
   copy_id // primary key, bigserial
   problem_id // foreign key, bigserial
   student_one_account // foreign key, 學生1學號(account) // 注意學生資料刪除時，須同步
   student_two_account // foreign key, 學生2學號(account) // 注意學生資料刪除時，須同步
   similarity // double, 兩者代碼相似度
   ```

4. **teacher** // 當刪除Teacher時，同步刪除Course

   ```json
   teacherId // primary key, bigserial
   account // text, 老師學號
   password // text, 老師密碼
   name // text, 老師密碼
   ```

5. **assistant**

   ```json
   assistantId // primary key, bigserial
   account // text, 助教學號
   password // text, 助教密碼
   name // text, 助教姓名
   ```

6. **admin**

   ```json
   adminId // primary key, bigserial
   account // text, 管理員學號
   password // text, 管理員密碼
   name // text, 管理員姓名
   ```

7. **course** // 當刪除course時，一併刪除problem、copy

   ```js
   course_id // primary key, bigserial
   teacher_id // foreign key, bigserial
   course_name // text, 課程名稱
   semester // text, 課程日期(格式:104上 || 104下)
   ```

8. **assistant_course** // 當刪除course時，須同步刪除assistant裡面的course

   ```json
   assistant_id // primary key, bigserial
   course_id // primary key, bigserial
   ```

9. **judge**

  ```json
  judge_id // primary key, bigserial
  problemId // foreign key, bigserial
  studentId // foreign key, bigserial
  rate // double, 題目難易度, 預設為0
  historyCode: {[ // json, 學生該題的歷史題交代碼
      "handDate": "", // String, 提交日期(格式:yy-mm-dd)
      "code": "", // String, 學生的代碼
      "runTime": 0, // double, 該代碼運行時間, 預設為0
      "output": [], // String, 輸出內容
      "error": [], // String, 錯誤訊息
      "symbol": [], // String, judge結果
      "score": 0, // double , 程式分數
  ]}
  ```

10. **feedback** // 當刪除學生會同步刪除feedback

  ```json
  feedbackId // primary key, bigserial
  courseId // foreign key, bigserial
  studentId // forign key, bigserial
  date // data, 提送feedback當時日期(格式yyyy-MM-dd)
  content // text, feedback內容
  ```

11. **dashboard**

    ```json
    dashboardId // primary key, bigserial
    onlineNum // integer, (今日)在線人數, 預設為0
    activeNum // integer, (今日)活躍人數，當天頁面訪問次數超過5(看checkLogin呼叫幾次), 預設為0
    todayDoDum // integer, 今日做題次數（今日批改次數), 預設為0
    weekDoNum // integer, 本週做題次數（本週批改次數）, 預設為0
    onlineData: {[ // json 在線與活躍人數線圖(上限：6天), 預設為空陣列
    	"date": "", // String, 日期(格式:yy-mm-dd)
         "onlineNum": 0, // integer, 日期當天在線人數
         "activeNum": 0 // integer, 日期當天活躍人數
    ]},
    doPerDayData: [{ // json 每日做題次數(上限：6天), 預設為空陣列
            "date": "", // String, 日期(格式:yyyy-mm-dd)
            "doPerDayNum": 0 // integer, 當日做題次數, 預設為0 
    }],
    judgeLiveData: [{ // json, 即時批改實況(上限：10則), 預設為空陣列
            "studentId": int, // String, 學生Id 
            "problemId": int, // String, 題目Id
    }]
    ```

12. **student_course** // course刪掉的話，學生須同步刪除裡面的course

    ```json
    student_id // primary key, bigserial
    course_id // primary key, bigserial
    ```


13. **group**

    ```js
    group_id // primary key, bigserial
    course_id // foreign key, bigserial
    member // text[], 學生的學號(account) // 刪除學生資料時，須同步刪除
    ```

    

# Restful API

- URL = https://hostname:8081/api/

- Response Format

  status：狀態碼

  msg：訊息

  result：回傳的資料

- Status Code

  | Status Code | Description |
  | ----------- | ----------- |
  | 200         | 請求成功    |
  | 404         | 請求失敗    |

1. 基礎api

   | API Method | API URL        | Desc         | Req Params        | Resp Result                                                  |
   | ---------- | -------------- | ------------ | ----------------- | ------------------------------------------------------------ |
   | POST       | URL/login      | 登入         | account, password | authority(判斷身份。student \|\| teacher \|\| assistant \|\| admin) |
   | POST       | URL/logout     | 登出         |                   |                                                              |
   | GET        | URL/checkLogin | 檢查登入狀態 |                   | status(boolean)、authority(判斷身份。 student \|\| teacher \|\| assistant \|\| admin) |

2. 學生api（student）

   - [x] 更改個人密碼
   - [x] 取得所有課程
   - [x] 取得課程下的個人學生資料
   - [x] 取得課程下的學生歷史成績及題目資訊
   - [x] 取得課程下的學生題目資訊(可根據題目類型、作答於否取得)
   - [x] 更新課程下的學生對題目的難易度評分

   | API Method | API URL                    | Desc                           | Req Params                                                   | Resp Result                                                  |
   | ---------- | -------------------------- | ------------------------------ | ------------------------------------------------------------ | :----------------------------------------------------------- |
   | POST       | URL/student/changePassword | 更改密碼                       | account, oriPassowrd, newPassword                            |                                                              |
   | GET        | URL/student/courseList     | 學生全部課程                   |                                                              | [{ courseId, courseName, teacherName, semester }]            |
   | GET        | URL/student/info           | 課程下的個人學生資料           | courseId                                                     | account, name, studentClass, undoNum, doneNum, bestCodeNum, correctNum, incorrectNum |
   | GET        | URL/student/historyScore   | 課程下的學生歷史成績及題目資訊 | courseId                                                     | [{problemId, problemName, type,  historyCode:[{handDate, code, runTime, output, symbol, errorMessage}], rate,  correctRate, isBestCode(Boolean), copys: [{account, similarity}]}] |
   | GET        | URL/student/problemInfo    | 課程下的學生所有題目資料       | courseId, type:(作業 \|\| 練習題 \|\| 討論題 \|\| 全部), isJudge(boolean) | [{problemId, name, type, deadline, rate}]                    |
   | POST       | URL/student/updateRate     | 課程下的學生對題目的難易度評分 | problemID, rate                                              |                                                              |

3. 題目api（problem）

   - [x] 取得單一題目資訊
   - [x] 取得課程下的所有題目資訊
   - [x] 新增、編輯、刪除題目

   | API Method | API URL                   | Desc                 | Req Params                                                   | Resp Result                                                  |
   | ---------- | ------------------------- | -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | GET        | URL/problem/getInfo       | 題目的資訊           | problemID                                                    | name, type, category, tag:[], rate, inputDesc, outputDesc, testcases: {inputSample, outputSample}, deadline, correctNum, incorrectNum, correctRate, bestStudentAccount |
   | GET        | URL/problem/getProblems   | 取得課程下的所有題目 | courseId                                                     | [{problemId, name, type, category, tag, status(判斷是否已過期。可作答、已關閉), undoStudentNum, doneStudentNum, rate, correctRate, deadline, bestStudentId, bestStudentName, bestRunTime, detectCopyResult: [{studentOneId, studentOneName, studentTwoId, studentTwoName, similarity}] (如果沒有就回空List)}] |
   | POST       | URL/problem/addProblem    | 新增題目             | courseId, name, type, category, tag:[], description, inputDesc, ouputDesc, testCases:{inputSample, outputSample}, deadline |                                                              |
   | POST       | URL/problem/editProblem   | 編輯題目             | problemId, name, type, category, tag:[], rate, description, inputDesc, ouputDesc, testCases:{inputSample, outputSample}, deadline |                                                              |
   | POST       | URL/problem/deleteProblem | 刪除題目             | problemId                                                    |                                                              |

4. 批改api（judge）

   - [x] 批改代碼
   - [x] 取得被批改後的資訊
   - [x] 檢查此題目是否被批改過
   - [x] 判斷抄襲

   | API Method | API URL               | Desc                   | Req Params                                  | Resp Result     |
   | ---------- | --------------------- | ---------------------- | ------------------------------------------- | --------------- |
   | POST       | URL/judge/judgeCode   | 批改代碼               | problemId, code, language(Java \|\| Python) |                 |
   | GET        | URL/judge/judgedInfo  | 已被批改後的資訊       | problemId                                   |                 |
   | GET        | URL/judge/checkJudged | 學生在此題是否已被批改 | porblemId                                   | judged(boolean) |
   | POST       | URL/judge/judgeCopy   | 判斷抄襲               | problemId                                   |                 |

5. 教授Api(teacher)

   - [x] 建立、刪除課程
   - [x] 將學生加入、退出課程
   - [x] 將助教加入、退出課程
   - [x] 新增、編輯、刪除題目(參考Problem Api)
   - [x] 取得所有授課課程資訊(參考Course Api)
   - [x] 取得課程下的所有題目資訊(參考Course Api)
   - [x] 取得課程下的所有學生資訊(參考Course Api)
   - [x] 取得課程下的所有feedback(參考Feedback Api)
   - [ ] 建立、刪除隊伍(參考Team Api)

   | **API Method** | API URL                       | Desc               | Req Params                       | Resp Result                          |
   | -------------- | ----------------------------- | ------------------ | -------------------------------- | ------------------------------------ |
   | POST           | URL/teacher/createCourse      | 建立課程           | courseName, semester             |                                      |
   | POST           | URL/teacher/deleteCourse      | 刪除課程           | courseId                         |                                      |
   | POST           | URL/teacher/addStudentList    | 將學生加入課程     | courseId, accountList: [account] |                                      |
   | POST           | URL/teacher/deleteStudentList | 將學生退出課程     | courseId, accountList: [account] |                                      |
   | POST           | URL/teacher/addAssistant      | 將助教加入課程     | courseId, accountList:[account]  |                                      |
   | POST           | URL/teacher/deleteAssistant   | 將助教退出課程     | courseId, accountList:[account]  |                                      |
   | GET            | URL/teacher/courseList        | 取得老師的所有課程 |                                  | [{ courseId, courseName, semester }] |

   

6. 助教api (assistant)

   - [x] 將學生加入、退出課程
   - [x] 新增、編輯、刪除題目(參考Problem Api)
   - [x] 取得課程下的所有feedback (參考Feedback Api)
   - [ ] 建立、刪除隊伍(參考 Team Api)

   | API Method | API URL                         | Desc           | Req Params                       | Resp Result |
   | ---------- | ------------------------------- | -------------- | -------------------------------- | ----------- |
   | POST       | URL/assistant/addStudentList    | 將學生加入課程 | courseId, accountList: [account] |             |
   | POST       | URL/assistant/deleteStudentList | 將學生退出課程 | courseId, accountList: [account] |             |

7. 分組api(team)(待做)

   - [ ] 建立、刪除隊伍

   | API Method | API URL               | Desc         | Req Params | Resp Result |
   | ---------- | --------------------- | ------------ | ---------- | ----------- |
   | POST       | URL/team/createGroups | 建立隨機隊伍 | courseId   |             |
   | POST       | URL/team/deleteGroups | 刪除隨機隊伍 | courseId   |             |

   

8. 課程api(course)

   - [x] 取得所有課程資訊
   - [x] 取得課程裡的學生所有資訊

   | API Method | API URL                    | Desc                     | Req Params | Resp Result                                                  |
   | ---------- | -------------------------- | ------------------------ | ---------- | ------------------------------------------------------------ |
   | GET        | URL/course/getCourses      | 取得所有課程             |            | [{ courseId, courseName, *teacherName, *semester }]          |
   | GET        | URL/course/getStudentInfo  | 取得課程裡的學生所有資訊 | courseId   |                                                              |
   | GET        | URL/course/getStudentsData | 取得課程的所有學生成績   | courseId   | studentID, studentName, studentClass, problems: [{name, score(若未被批改回傳"未作答"), date, type, historyCode}] |

   

9. 排行api (rank)

   - [x] 取得正確解題的學生排行
   - [x] 取得最佳解答的學生排行

   | API Method | API URL                  | Desc                                                    | Req Params | Resp Result                          |
   | ---------- | ------------------------ | ------------------------------------------------------- | ---------- | ------------------------------------ |
   | GET        | URL/rank/getCorrectRank  | 取得正確解題的學生排行（前五名，有同值則同名，且值>0）  | courseId   | [{rank, account, name, correctNum}]  |
   | GET        | URL/rank/getBestCodeRank | 取得最佳解答的學生排行 （前五名，有同值則同名，且值>0） | courseId   | [{rank, account, name, bestCodeNum}] |

   

10. 回饋Api(feedback)

  - [x] 新增課程下的回饋
  - [x] 取得課程下的所有回饋

  | API Method | API URL                         | Desc                     | Req Params        | Resp Result |
  | ---------- | ------------------------------- | ------------------------ | ----------------- | ----------- |
  | POST       | URL/feedback/addFeedback        | 新增回饋                 | courseId, content |             |
  | GET        | URL/feedback/getCourseFeedbacks | 取得課程下的所有feedback | courseId          |             |

  

11. 管理員api(admin)(待做)

    | API Method | API URL | Desc | Req Params | Resp Result |
    | ---------- | ------- | ---- | ---------- | ----------- |
    |            |         |      |            |             |

    

12. 數據api (data)(待做)

    | API Method | API URL                   | Desc                                                         | Req Params | Resp Result                                                  |
    | ---------- | ------------------------- | ------------------------------------------------------------ | ---------- | ------------------------------------------------------------ |
    | GET        | URL/data/getBasicData     | 取得在線人數(onlineNum)、活躍人數(activeNum)、今日做題次數(todayDoNum)、本週做題次數(WeekDoNum)的資訊 |            | onlineNum, activeNum, todayDoNum,  weekDoNum                 |
    | GET        | URL/data/getOnlineData    | 取得在線與活躍人數統計（6天）                                |            | [{date, onlineNum, activeNum}]                               |
    | GET        | URL/data/getDoPerDayData  | 取得每日做題人數（6天）                                      |            | [{date, doPerDayNum}]                                        |
    | GET        | URL/data/getJudgeLiveData | 取得批改即時資訊                                             |            | [{time, studentID, studentName,  problemID, problemName, score, errorMsg}] |

    