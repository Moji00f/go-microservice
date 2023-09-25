# go-microservice

### install packe
```
go get github.com/go-chi/chi/v5
go get github.com/go-chi/chi/v5/middleware 
go get github.com/go-chi/cors

#models
golang.org/x/crypto/bcrypt

# connect to database
go get github.com/jackc/pgconn
go get github.com/jackc/pgx/v4/stdlib 

# connect to mongodb
go get go.mongodb.org/mongo-driver/mongo
go get go.mongodb.org/mongo-driver/options

# Email
go get github.com/vanng822/go-premailer/premailer
go get github.com/xhit/go-simple-mail/v2

# RabbitMQ
go get github.com/rabbitmq/amqp091-go

#grpc
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.27 
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2 
wget https://github.com/protocolbuffers/protobuf/releases/download/v24.3/protoc-24.3-linux-x86_64.zip
unzip 
cd logs
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative logs.proto 

go get google.golang.org/grpc 
go get google.golang.org/protobuf
```