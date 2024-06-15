# GraphQL (Graph Query Language)


## REST API VS GraphQL 
- RESTFul API 메소드는 HTTP 메소드 + URI 로 자원을 구분했음
  - 요청의 내용을 직관적으로 파악하기 쉬움
  - 많은 엔드포인트를 관리해야 함
  - 어느 자원에 대한 응답을 받고 싶어하는지는 요청할 수 있지만, 어떤 항목을 가져올 것인지는 특정하기 어려움
    - `/v1/student/1` -> 로 요청했을 때, 학생의 이름만 원하는 경우에도 서버에서 보내는 모든 리소스 데이터들이 반환 -> overfetching
    - 데이터 낭비, 오버헤드 발생 가눙성
- GrpahQL은 SOAP처럼 하나의 엔드포인트를 사용
  - 일반적으로 모든 요청을 POST요청으로 보냄
  - 딱 쿼리에서 요청한 데이터(클라이언트가 원하는 데이터)만 받을 수 있음 -> REST API 의 오버페칭 해결
  - RESTful API의 underfetching : 여러번 API를 호출해야하는 문제 해결
    - ex. 학생 정보, 학생의 성적, 등등을 각각 호출해야함
    - 하나의 요청으로는 충분한 데이터를 얻기 어려움
  - 구독 (Subscription)
    - RESTful API에서는 변경사항을 알기 위해 서버에 수시로 요청을 보내야 함
    - 웹 소켓을 사용
  - 단점
    - 캐싱의 어려움
    - REST API에 비해 서버에 부담
    - 높은 러닝 커브
  - 적합한 서비스
    - 복잡하고 방대한 데이터를 가진 GraphQL
    - 클라이언트가 데이터 요청 제어권이 많은 서비스
    - 실시간 업데이트 감지가 필요한 서비스


### 학생정보를 얻어오는 요청
``` node.js
const fetch = require('node-fetch');

const query = `
query GetStudent($studentId: Int!) {
  student(studentId: $studentId) {
    studentName
    studentId
    grade
    major
  }
}`;

const variables = {
  studentId: 1
};

fetch('http://localhost:4000', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    query,
    variables,
  }),
})
  .then(res => res.json())
  .then(res => console.log(res.data))
  .catch(error => console.error(error));

```
``` json
{
  "data": {
    "student": {
      "studentName": "John Doe",
      "studentId": 1,
      "grade": 3,
      "major": "Computer Science"
    }
  }
}
```

### n학생의 정보를 요청할 때 학생의 정보와 함께 현재 수강 정보와 성적 정보를 포함한 GraphQL API
``` node.js
// Resolvers define the technique for fetching the types defined in the schema
const resolvers = {
    Query: {
        student: (_, { studentId }) => {
            const student = students[studentId];
            if (!student) return null;
            return {
                ...student,
                courses: courses[studentId] || [],
                grades: grades[studentId] || []
            };
        }
    }
};

``` node.js
const fetch = require('node-fetch');

const query = `
query GetStudent($studentId: Int!) {
  student(studentId: $studentId) {
    studentName
    studentId
    grade
    major
    courses {
      courseId
      courseName
      instructor
    }
    grades {
      courseId
      courseName
      grade
    }
  }
}`;

const variables = {
  studentId: 1
};

fetch('http://localhost:4000', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    query,
    variables,
  }),
})
  .then(res => res.json())
  .then(res => console.log(JSON.stringify(res.data, null, 2)))
  .catch(error => console.error(error));

```
``` json
{
  "data": {
    "student": {
      "studentName": "John Doe",
      "studentId": 1,
      "grade": 3,
      "major": "Computer Science",
      "courses": [
        {
          "courseId": 101,
          "courseName": "Algorithms",
          "instructor": "Dr. Brown"
        },
        {
          "courseId": 102,
          "courseName": "Data Structures",
          "instructor": "Dr. Green"
        }
      ],
      "grades": [
        {
          "courseId": 101,
          "courseName": "Algorithms",
          "grade": "A"
        },
        {
          "courseId": 102,
          "courseName": "Data Structures",
          "grade": "B+"
        }
      ]
    }
  }
}
```
