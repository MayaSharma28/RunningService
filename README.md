Feature: Validate Affiliate Account Info Service

  Background:
    * Base URI is set
    * Headers like Authorization, Correlation-ID, and Client-ID are configured

  Scenario: Valid account ID with valid token and headers should return 200 OK
    Given the account ID is "123456"
    And a valid token is used
    When the user sends the request with all required headers and query params
    Then the response status should be 200
    And the account info should contain the ID "123456"

  Scenario: Valid account ID with invalid token should return 403 or 401
    Given the account ID is "123456"
    And an invalid token is used
    When the user sends the request with all required headers and query params
    Then the response status should be 401 or 403

  Scenario: Invalid account ID should return 404
    Given the account ID is "nonexistent-id"
    And a valid token is used
    When the user sends the request with all required headers and query params
    Then the response status should be 404

  Scenario: Missing required headers should return 400 or 403
    Given the account ID is "123456"
    And a valid token is used
    When the user sends the request without required headers
    Then the response status should be 400 or 403

  Scenario: Valid account ID with pagination query params
    Given the account ID is "123456"
    And a valid token is used
    When the user sends the request with all required headers and pagination params
    Then the response status should be 200
    And the result count should be less than or equal to 3

import io.cucumber.java.en.*;
import io.restassured.RestAssured;
import io.restassured.response.Response;

import static org.testng.Assert.*;

import java.util.HashMap;
import java.util.Map;

public class AffiliateAccountSteps {

    private String baseUri = "https://<your-host>";
    private String accountId;
    private String token;
    private Map<String, String> headers = new HashMap<>();
    private Response response;

    @Given("the account ID is {string}")
    public void the_account_ID_is(String id) {
        this.accountId = id;
    }

    @And("a valid token is used")
    public void a_valid_token_is_used() {
        this.token = "Bearer valid-access-token";
    }

    @And("an invalid token is used")
    public void an_invalid_token_is_used() {
        this.token = "Bearer unauthorized-token";
    }

    @When("the user sends the request with all required headers and query params")
    public void the_user_sends_request_with_all_required_headers() {
        headers.clear();
        headers.put("Authorization", token);
        headers.put("X-Correlation-ID", "corr-7890");
        headers.put("X-Client-ID", "client-1234");

        response = RestAssured
                .given()
                    .baseUri(baseUri)
                    .basePath("/affiliate/account/info")
                    .queryParam("id", accountId)
                    .queryParam("limit", 3)
                    .queryParam("pageSize", 3)
                    .headers(headers)
                .when()
                    .get();
    }

    @When("the user sends the request without required headers")
    public void the_user_sends_request_without_required_headers() {
        headers.clear();
        headers.put("Authorization", token); // Missing correlation ID and client ID

        response = RestAssured
                .given()
                    .baseUri(baseUri)
                    .basePath("/affiliate/account/info")
                    .queryParam("id", accountId)
                    .headers(headers)
                .when()
                    .get();
    }

    @When("the user sends the request with all required headers and pagination params")
    public void the_user_sends_request_with_pagination() {
        headers.clear();
        headers.put("Authorization", token);
        headers.put("X-Correlation-ID", "corr-7890");
        headers.put("X-Client-ID", "client-1234");

        response = RestAssured
                .given()
                    .baseUri(baseUri)
                    .basePath("/affiliate/account/info")
                    .queryParam("id", accountId)
                    .queryParam("limit", 3)
                    .queryParam("pageSize", 3)
                    .headers(headers)
                .when()
                    .get();
    }

    @Then("the response status should be {int}")
    public void the_response_status_should_be(int expectedStatus) {
        assertEquals(response.statusCode(), expectedStatus);
    }

    @Then("the response status should be 401 or 403")
    public void the_response_status_should_be_401_or_403() {
        int status = response.statusCode();
        assertTrue(status == 401 || status == 403, "Expected 401 or 403, but got: " + status);
    }

    @Then("the response status should be 400 or 403")
    public void the_response_status_should_be_400_or_403() {
        int status = response.statusCode();
        assertTrue(status == 400 || status == 403, "Expected 400 or 403, but got: " + status);
    }

    @And("the account info should contain the ID {string}")
    public void the_account_info_should_contain_the_ID(String expectedId) {
        String returnedId = response.jsonPath().getString("accountId");
        assertEquals(returnedId, expectedId);
    }

    @And("the result count should be less than or equal to {int}")
    public void the_result_count_should_be_less_than_or_equal_to(int maxCount) {
        int size = response.jsonPath().getList("data").size();
        assertTrue(size <= maxCount, "Returned result count should be <= " + maxCount);
    }
}
    
