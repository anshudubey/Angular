# Angular
new line

 Angular 17 automatically removes component-associated styles from the DOM when the component is destroyed. In Angular 15, these styles persisted, which some applications might have relied on implicitly.

**************************************************************************
Install dependencies in each app (npm install or yarn).
Start apps in separate terminals:
# Angular Microfrontend
cd angular-app
ng serve --port 4201

# React Microfrontend
cd react-app
npx webpack serve

# Container Host App
cd container-app
ng serve --port 4200


******************************************************************************
In Service B (Delete API):
@DeleteMapping("/cleanup/delete")
public ResponseEntity<DeleteStatusResponse> deleteOldRecords() {
    DeleteStatusResponse result = deletionService.deleteRecords();
    return ResponseEntity.ok(result);
}

In Service B – DeletionService:
public DeleteStatusResponse deleteRecords() {
    DeleteStatusResponse response = new DeleteStatusResponse();

    try {
        table1Repo.deleteOldRecords(); // or all records
        response.setTable1Status("SUCCESS");
    } catch (Exception ex) {
        response.setTable1Status("FAILURE");
        response.setTable1Error(ex.getMessage());
    }

    try {
        table2Repo.deleteOldRecords(); // or all records
        response.setTable2Status("SUCCESS");
    } catch (Exception ex) {
        response.setTable2Status("FAILURE");
        response.setTable2Error(ex.getMessage());
    }

    return response;
}

In Service A (Scheduler):
Use RestTemplate.exchange() with HttpMethod.DELETE because RestTemplate does not have deleteForObject() that returns a body.

ResponseEntity<DeleteStatusResponse> response = restTemplate.exchange(
    "http://service-b/cleanup/delete",
    HttpMethod.DELETE,
    null,
    DeleteStatusResponse.class
);

DeleteStatusResponse result = response.getBody();

public void callDeletionService(String entityId) {
    try {
        DeleteRequest req = new DeleteRequest(entityId);
        DeleteStatusResponse res = restTemplate.postForObject(
            "http://service-b/cleanup/delete", req, DeleteStatusResponse.class);

        // Update history table with each status
        historyRepo.save(new History(entityId, "TABLE1", res.getTable1Status(), res.getTable1Error()));
        historyRepo.save(new History(entityId, "TABLE2", res.getTable2Status(), res.getTable2Error()));

    } catch (Exception ex) {
        historyRepo.save(new History(entityId, "SERVICE_B", "FAILURE", "Service B not reachable: " + ex.getMessage()));
    }
}

1. Service B - Response Model

public class DeleteStatusResponse {
    private String table1Status;
    private String table1Error;

    private String table2Status;
    private String table2Error;

    // Getters, Setters
}
********************************************************************************************************************************************************************
Service B
@DeleteMapping("/cleanup/delete")
public ResponseEntity<Map<String, String>> deleteRecords() {
    Map<String, String> result = deletionService.deleteRecords();
    return ResponseEntity.ok(result);
}


@Service
public class DeletionService {

    @Autowired private Table1Repository table1Repo;
    @Autowired private Table2Repository table2Repo;

    public Map<String, String> deleteRecords() {
        Map<String, String> result = new HashMap<>();

        try {
            table1Repo.deleteAll(); // or your custom delete logic
            result.put("table1", "SUCCESS");
        } catch (Exception e) {
            result.put("table1", "FAILURE: " + e.getMessage());
        }

        try {
            table2Repo.deleteAll();
            result.put("table2", "SUCCESS");
        } catch (Exception e) {
            result.put("table2", "FAILURE: " + e.getMessage());
        }

        return result;
    }
}

Service A
public Map<String, String> invokeDeleteService() {
    try {
        ResponseEntity<Map> response = restTemplate.exchange(
            "http://service-b/cleanup/delete",
            HttpMethod.DELETE,
            null,
            Map.class
        );
        return response.getBody();
    } catch (Exception ex) {
        Map<String, String> errorMap = new HashMap<>();
        errorMap.put("SERVICE_B", "FAILURE: " + ex.getMessage());
        return errorMap;
    }
}


public void processAndSaveHistory(String jobId, Map<String, String> statusMap) {
    for (Map.Entry<String, String> entry : statusMap.entrySet()) {
        String table = entry.getKey();
        String status = entry.getValue();

        // Example: Save to your history table
        History history = new History();
        history.setJobId(jobId);  // or entityId / schedulerId
        history.setComponent(table);
        history.setStatus(status);
        history.setTimestamp(LocalDateTime.now());

        historyRepository.save(history);
    }
}

 Final Usage Flow
public void scheduledCleanupJob() {
    String jobId = UUID.randomUUID().toString();  // optional tracking ID
    Map<String, String> result = invokeDeleteService();
    processAndSaveHistory(jobId, result);
}


*********************************************************************************************************
public Map<String, String> deleteRecords() {
    Map<String, String> response = new HashMap<>();
    boolean failed = false;
    StringBuilder error = new StringBuilder();

    try {
        table1Repo.deleteAll();
    } catch (Exception ex) {
        failed = true;
        error.append(ex.getMessage());
    }

    try {
        table2Repo.deleteAll();
    } catch (Exception ex) {
        failed = true;
        if (error.length() > 0) error.append(" | ");  // separate multiple errors
        error.append(ex.getMessage());
    }

    if (failed) {
        response.put("status", "FAIL");
        response.put("error", error.toString());
    } else {
        response.put("status", "SUCCESS");
    }

    return response;
}




