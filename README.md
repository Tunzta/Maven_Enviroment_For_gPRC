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

package GRPC;

option java_package = "GRPC";
option java_multiple_files = true;

service JudgeService {
  rpc Request (JudgeRequest) returns (JudgeResponse);
  rpc Submit  (SubmitRequest) returns (SubmitResponse);
}

message JudgeRequest {
  string student_code    = 1;
  string question_alias  = 2;
}

message JudgeResponse {
  string request_id = 1;
  string data       = 2;
}

message SubmitRequest {
  string student_code    = 1;
  string question_alias  = 2;
  string request_id      = 3;
  string answer          = 4;
}

message SubmitResponse {
  string status  = 1;
  string message = 2;
}
```
