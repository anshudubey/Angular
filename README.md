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

