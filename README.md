# gRPC Client Setup

## 1. `pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>grpc-client</artifactId>
    <version>1.0</version>

    <properties>
        <maven.compiler.release>21</maven.compiler.release>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>

        <grpc.version>1.81.0</grpc.version>
        <protobuf.version>3.25.9</protobuf.version>

        <fixed.os.classifier>windows-x86_64</fixed.os.classifier>

        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-netty-shaded</artifactId>
            <version>${grpc.version}</version>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-protobuf</artifactId>
            <version>${grpc.version}</version>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-stub</artifactId>
            <version>${grpc.version}</version>
        </dependency>

        <dependency>
            <groupId>com.google.protobuf</groupId>
            <artifactId>protoc</artifactId>
            <version>${protobuf.version}</version>
            <classifier>${fixed.os.classifier}</classifier>
            <type>exe</type>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>protoc-gen-grpc-java</artifactId>
            <version>${grpc.version}</version>
            <classifier>${fixed.os.classifier}</classifier>
            <type>exe</type>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.21.3</version>
        </dependency>
        <dependency>
            <groupId>javax.annotation</groupId>
            <artifactId>javax.annotation-api</artifactId>
            <version>1.3.2</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <version>0.6.1</version>
                <configuration>
                    <protocArtifact>
                        com.google.protobuf:protoc:${protobuf.version}:exe:${fixed.os.classifier}
                    </protocArtifact>
                    <pluginId>grpc-java</pluginId>
                    <pluginArtifact>
                        io.grpc:protoc-gen-grpc-java:${grpc.version}:exe:${fixed.os.classifier}
                    </pluginArtifact>
                    <outputDirectory>
                        ${project.basedir}/src/main/java
                    </outputDirectory>
                    <clearOutputDirectory>false</clearOutputDirectory>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>compile-custom</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## 2. `judge.proto`

```proto
syntax = "proto3";

option java_multiple_files = true;

package GRPC;

// Backward-compatible string-based judging service.
service JudgeService {
  rpc Request (JudgeRequest) returns (JudgeResponse);
  rpc Submit (SubmitRequest) returns (SubmitResponse);
}

message JudgeRequest {
  string student_code = 1;
  string question_alias = 2;
}

message JudgeResponse {
  string request_id = 1;
  string data = 2;
}

message SubmitRequest {
  string student_code = 1;
  string question_alias = 2;
  string request_id = 3;
  string answer = 4;
}

message SubmitResponse {
  string status = 1;
  string message = 2;
}

// Typed service for advanced gRPC questions. It runs in parallel with
// JudgeService so existing Q2111-Q2131 clients keep working unchanged.
service TypedJudgeService {
  rpc RequestTyped (TypedJudgeRequest) returns (TypedJudgeResponse);
  rpc SubmitTyped (TypedSubmitRequest) returns (TypedSubmitResponse);
}

message TypedJudgeRequest {
  string student_code = 1;
  string question_alias = 2;
}

message TypedJudgeResponse {
  string request_id = 1;
  oneof data {
    TransactionRiskBatchData transaction_risk_batch = 10;
    SensorTelemetryData sensor_telemetry = 11;
    TextBatchData text_batch = 12;
    ShippingQuoteData shipping_quote = 13;
    EnrollmentData enrollment = 14;
  }
}

message TypedSubmitRequest {
  string student_code = 1;
  string question_alias = 2;
  string request_id = 3;
  oneof answer {
    TransactionRiskAnswer transaction_risk_answer = 10;
    SensorTelemetryAnswer sensor_telemetry_answer = 11;
    TextBatchAnswer text_batch_answer = 12;
    ShippingQuoteAnswer shipping_quote_answer = 13;
    EnrollmentAnswer enrollment_answer = 14;
  }
}

message TypedSubmitResponse {
  string status = 1;
  string message = 2;
}

message TransactionRecord {
  string transaction_id = 1;
  double amount = 2;
  string currency = 3;
  string country = 4;
  int32 chargeback_count = 5;
  bool new_device = 6;
}

message TransactionRiskBatchData {
  repeated TransactionRecord transactions = 1;
}

message TransactionRiskAnswer {
  repeated string high_risk_transaction_ids = 1;
  int32 review_count = 2;
  double total_high_risk_amount = 3;
}

message SensorReading {
  string sensor_id = 1;
  int64 timestamp_epoch_seconds = 2;
  double value = 3;
}

message SensorTelemetryData {
  repeated SensorReading readings = 1;
  double threshold = 2;
}

message SensorTelemetryAnswer {
  double average = 1;
  double p95 = 2;
  int32 anomaly_count = 3;
}

message TextBatchData {
  repeated string entries = 1;
  string mode = 2;
}

message TextBatchAnswer {
  repeated string values = 1;
  map<string, int32> counts = 2;
}

message CarrierQuote {
  string carrier = 1;
  double base_fee = 2;
  double per_kg_fee = 3;
  int32 eta_days = 4;
  double reliability = 5;
}

message ShippingQuoteData {
  string order_id = 1;
  double weight_kg = 2;
  int32 max_eta_days = 3;
  repeated CarrierQuote quotes = 4;
}

message ShippingQuoteAnswer {
  string carrier = 1;
  double total_fee = 2;
  int32 eta_days = 3;
}

message EnrollmentData {
  string student_id = 1;
  repeated string completed_courses = 2;
  repeated string required_courses = 3;
  double gpa = 4;
  double min_gpa = 5;
}

message EnrollmentAnswer {
  bool eligible = 1;
  repeated string missing_courses = 2;
  double gpa_gap = 3;
}
```
