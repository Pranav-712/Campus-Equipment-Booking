# Campus Equipment Booking Service
Java 17 / Spring Boot REST API for a CI/CD lab assessment.

## Build and testsssss
`mvn clean test`

## Run
`mvn spring-boot:run` — port 8080.

## Endpoints
- `GET /api/equipment`
- `GET /api/equipment/{id}`
- `POST /api/bookings`
- `GET /api/bookings/{id}`
- `POST /api/bookings/{id}/cancel`
- `GET /actuator/health`

Inventory is in memory and resets on restart. No database is required.



## Docker Deployment

Build the application:

```bash
mvn clean test
mvn clean package
```

Build and run the Docker image:

```bash
docker build -t campus-equipment-booking:set-d .
docker run -d --name campus-equipment-booking-container -p 8082:8080 campus-equipment-booking:set-d
```

Health check: `http://localhost:8082/actuator/health`

## Jenkins CI/CD

The Jenkins pipeline checks out the GitHub repository, runs Maven tests, packages the JAR, builds the Docker image, deploys the container, and checks application health.

A GitHub push webhook triggers the Jenkins pipeline automatically.

## Kubernetes Deployment

Load the image into Minikube:

```bash
minikube image load campus-equipment-booking:set-d
```

Deploy the application:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

Verify and scale:

```bash
kubectl get deployments
kubectl get pods
kubectl scale deployment campus-equipment-booking --replicas=2
kubectl rollout status deployment/campus-equipment-booking
kubectl port-forward service/campus-equipment-booking 8083:8080
```

Health check: `http://localhost:8083/actuator/health`

## Change Request

Fixed booking capacity validation so requests cannot exceed available inventory. Added tests for exact-capacity bookings, overbooking, negative quantities, and invalid borrower IDs. Rejected bookings must not reduce inventory or create a confirmed booking.
