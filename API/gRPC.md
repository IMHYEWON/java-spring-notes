## RPC란?
- Remote Produce Call
- 언어 상관없이 다른 프로그램의 기능을 자기 기능인 것처럼 사용하는 것
- 자신의 함수를 사용하든 다른 프로그램의 함수를 사용
- STUB : stub객체를 통해 원격의 서버에 시킴
- 개발자는 서버나 통신과정을 고려할 필요없이 자신의 기능들을 필요에 따라 사용할 수 있게 되는것

# gRPC
- google Remote Produce Call
- google에서 만든 RPC 라이브러리

# gRPC VS REST API
- 요즘 많은 날의 서비스들이 MSA 구조
- 이 사이에서 여러 서비스들이 서로 클라이언트, 서버가 됨
- REST API 에서는 json을 요청/응답으로 사용됨.
  - xml보다는 간단하지만, 중복되는 부분 = key가 낭비됨
  - 그렇지만, key 가 없으면 각 필드가 무엇을 의미하는지 알기 힘듬
  - -> **그렇기에 gRPC는 프로토콜 버퍼**를 사용
 

## gRPC의 proto
- 클라이언트와 서버가 메시지를 어떻게 주고 받을지 정하는 약속 -> 프로토콜
- 프로토콜 버퍼는 이미 정해진 약속을 기반으로 통신하기 때문에 메시지키를 간소화
- 바이너리 형태로 직렬화해서 전송하기 때문에 JSON과 같은 텍스트 기반의 요청보다 용량이 작음
- HTTP/2 기반 : 한번 커넥션이 생기면 클라이언트와 서버는 동시에 메시지를 보낼 수 도 있고, 서버가 클라이언트 요청 순서와 별개로 능동적으로 응답을 주고 받을 수 있음
  - Restful API 에서 자주사용되는 HTTP1.1
- TLS를 통해 데이터를 암호화해서 전송
- 브라우저에서 동작하는 웹 프론트엔드에서는 거의 동작하지 않음 -> 지원이 부족
- 가장 많이 사용되는 곳은 MSA간 통신, 모바일 어플리케이션, 게임서버 등

``` bash
python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. student.proto
```
``` gRPC
syntax = "proto3";

package student;

// Student request message
message StudentRequest {
    string student_name = 1;
    int32 student_id = 2;
}

// Student response message
message StudentResponse {
    string student_name = 1;
    int32 student_id = 2;
    int32 grade = 3;
    string major = 4;
}

// Student service definition
service StudentService {
    // Sends a request with student name and ID and receives student information
    rpc GetStudentInfo (StudentRequest) returns (StudentResponse);
}

```

``` python
import grpc
import student_pb2
import student_pb2_grpc

def run():
    with grpc.insecure_channel('localhost:50051') as channel:
        stub = student_pb2_grpc.StudentServiceStub(channel)
        response = stub.GetStudentInfo(student_pb2.StudentRequest(student_name='John Doe', student_id=1))
        print(f"Student Name: {response.student_name}")
        print(f"Student ID: {response.student_id}")
        print(f"Grade: {response.grade}")
        print(f"Major: {response.major}")

if __name__ == '__main__':
    run()

```

``` bash
protoc --java_out=src/main/java --grpc-java_out=src/main/java -I=. student.proto
```
``` java
import io.grpc.Server;
import io.grpc.ServerBuilder;
import io.grpc.stub.StreamObserver;
import student.StudentServiceGrpc;
import student.StudentServiceOuterClass;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

public class StudentServer {
    private Server server;

    public static void main(String[] args) throws IOException, InterruptedException {
        final StudentServer server = new StudentServer();
        server.start();
        server.blockUntilShutdown();
    }

    private void start() throws IOException {
        int port = 50051;
        server = ServerBuilder.forPort(port)
                .addService(new StudentServiceImpl())
                .build()
                .start();
        System.out.println("Server started, listening on " + port);
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            System.err.println("*** shutting down gRPC server since JVM is shutting down");
            StudentServer.this.stop();
            System.err.println("*** server shut down");
        }));
    }

    private void stop() {
        if (server != null) {
            server.shutdown();
        }
    }

    private void blockUntilShutdown() throws InterruptedException {
        if (server != null) {
            server.awaitTermination();
        }
    }

    static class StudentServiceImpl extends StudentServiceGrpc.StudentServiceImplBase {
        private static Map<Integer, StudentInfo> studentInfoMap = new HashMap<>();

        static {
            studentInfoMap.put(1, new StudentInfo("John Doe", 3, "Computer Science"));
            studentInfoMap.put(2, new StudentInfo("Jane Smith", 2, "Mathematics"));
        }

        @Override
        public void getStudentInfo(StudentServiceOuterClass.StudentRequest request,
                                   StreamObserver<StudentServiceOuterClass.StudentResponse> responseObserver) {
            StudentInfo info = studentInfoMap.getOrDefault(request.getStudentId(),
                    new StudentInfo("Unknown", 0, "Unknown"));

            StudentServiceOuterClass.StudentResponse response = StudentServiceOuterClass.StudentResponse.newBuilder()
                    .setStudentName(info.name)
                    .setStudentId(request.getStudentId())
                    .setGrade(info.grade)
                    .setMajor(info.major)
                    .build();

            responseObserver.onNext(response);
            responseObserver.onCompleted();
        }
    }

    static class StudentInfo {
        String name;
        int grade;
        String major;

        StudentInfo(String name, int grade, String major) {
            this.name = name;
            this.grade = grade;
            this.major = major;
        }
    }
}

```
