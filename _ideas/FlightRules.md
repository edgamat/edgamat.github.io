## Flight Rules

https://www.nasa.gov/centers/johnson/pdf/514212main_SJ-02_Overview_Flight_Rules.pdf


Flight Rules are the hard-earned body of knowledge recorded in manuals that list, step by step, what to do it X occurs and why. Essentially, they are extremely detailed, scenario-specific standard operating procedures.

Flight Rules protest against the temptation to take risks, which is strongest when momentum has been building to meet a deadline.

### Standard Operating Procedures

101) When making a change to the code
102) When fixing a bug
103) When creating a Pull Request
104) When adding a feature not ready for production
105) When adding an endpoint
106) When adding a message consumer
107) When adding a message producer
108) When adding a log entry
109) When handling an exception
110) When adding an application setting
111) When modifying an application setting
112) When adding a health check
113) When adding a new table/view to an EF Core context (not owned by the project)
114) When adding a call to a stored procedure (not owned by the project)

### Deployment Procedures

201) When a build fails in DEV

202) When a deployment fails in DEV

203) When a deployment fails in TEST

204) When a nightly integration test fails

205) When a nightly security scan fails

206) When a deployment fails in PROD

### Data Migration Rules

1. Each data migration script should be idempotent. Running a data migration script multiple times should do no harm and, whenever possible, generating no errors in the logs. Note: The default scripts generated from EF Core Entity Framework are not something we are in control of. However, if we add custom scripts to add data or manipulate existing database objects, we must ensure they are idempotent.

2. Each application should ensure that data migration scripts are only run by the primary instance of the application. In other words, an application must not attempt to run the migration script from multiple instances simultaneously. This can be accomplished by using Eureka to determine if the instance is the primary instance prior to running a migration script.

3. Data migration scripts that modify the structure of existing database objects should not be run in Production until it is verified that the migration has run successfully in the Prod Fix Environment. We want to make sure the migration script works properly with the database loaded with data from Production.

4. Data migration scripts that modify the structure of existing database objects should not be run in Production until it is verified that the Production database has a known good backup available. We want to make sure that if something goes wrong, there is a means to restore the affected database.

5. When a data migration is added to an application, the developer should add a new Task to the associated story in JIRA called "Verify Data Migration" and assign the task to whomever is responsible to deploying the code into Production (usually Mike Michaud). This is the cue to Mike that there will be a data migration as part of the next deployment.

6. When a data migration applies to tables not owned by our applications, like the suite of AIM databases, a copy of the migration script should be added to the source code repository (e.g. manual-scripts) in order for developers to apply the same change to the local copy they have of the database.

7. When using EF Core migrations, developers should decorate the data migration partial class with the `[ExcludeFromCodeCoverage]` attribute to exclude these auto-generated classes from influencing the code coverage metrics for the application.

### Pull Request Rules

1. The author of the Pull Request should verify that the target environment is ready to accept the new functionality prior to completing the Pull Request. For example, if a database object needs to be added/modified in the CIS database, the author should verify the DEV/TEST instance of CIS has the updates applied.

2. The reviewer should verify that the application version (`version.ps1`) is unique, following the Semantic Versioning rules.

3. The reviewer should verify that any data migration scripts in the Pull Request are idempotent.

4. The reviewer should checkout the code branch and verify that 
    - the code compiles,
    - all unit tests pass,
    - no un-necessary compiler warnings have been added,
    - the code changes adhere to our agreed-upon coding standards

5. The reviewer should verify that the private build of the Pull Request branch successful completed in TeamCity.

### Developer Testing Rules

The purpose of the Developer Testing task on Sprint Stories is to:
    - verify that the updated application is deployed correctly into the Development Environment
    - review the application logs, message queues, etc. for abnormalities
    - verify that any data migrations were applied successfully
    - verify that the CI/CD Integration Tests in TeamCity all passed


1. The developer should verify that the code deployed correctly into the Development Environment (Build/Test step was successful and the Deploy step was successful).

2. The developer should verify that all re-seed scripts were successfully applied to the Development Environment. This should also be done in the Test Environment if the application is automatically deployed to the Test Environment. 

3. If the application is automatically deployed to the Test Environment, the developer should verify that the code deployed correctly into the Test Environment.

4. The developer should review the application logs, message queues, etc. for abnormalities after the new version of the application is deployed into the Development Environment. 

5. The developer should verify that all data migrations were successfully applied after the new version of the application is deployed into the Development Environment. This should also be done in the Test Environment if the application is automatically deployed to the Test Environment. 

6. The developer should verify that the CI/CD Integration Tests in TeamCity all passed after the new version of the application is deployed into the Development Environment. 

7. The developer should post messages to the team when there is a failed deployment and when the deployment is corrected. The messages should indicate any impacts the failed deployment may have on users and if possible an estimate on how long it will take to get things corrected.

8. The developer should verify each of the acceptance criteria is satisfied in the Development Environment, or the lowest environment where verifying the behavior is possible (for example, some tests might only be possible in the Test Environment).

9. The developer should document the steps, including any JSON payloads, SQL scripts, etc., in the Developer Testing task on each story.

10. The developer should post a message to the team once they have verified that Automated QA testing can start.

11. Once all developer testing is complete, the developer should request the deployment of the application to the Test Environment. This does not apply to applications that are automatically deployed to the Test Environment. The developer should verify that the code deployed correctly into the Test Environment.

12. The developer should post a message to the team once they have verified that the application is successfully deployed to the Test Environment and Business Testing can start. 



