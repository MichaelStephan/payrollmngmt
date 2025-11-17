# Tomcat Upgrade Instructions - CVE-2025-55752 Remediation

## Quick Summary

The `payrollmngmt` application has been updated to remediate **CVE-2025-55752** (HIGH severity Relative Path Traversal vulnerability) by upgrading Apache Tomcat from the vulnerable version to **9.0.109**.

## What Changed?

### File Modified: `pom.xml`

Two changes were made:

1. **Added Tomcat version override** in the properties section (line 18):
   ```xml
   <tomcat.version>9.0.109</tomcat.version>
   ```

2. **Added explicit dependency management** (lines 68-87) to ensure all Tomcat components are upgraded:
   ```xml
   <dependencyManagement>
       <dependencies>
           <dependency>
               <groupId>org.apache.tomcat.embed</groupId>
               <artifactId>tomcat-embed-core</artifactId>
               <version>${tomcat.version}</version>
           </dependency>
           <!-- Additional Tomcat dependencies -->
       </dependencies>
   </dependencyManagement>
   ```

## How to Deploy

### Step 1: Pull the Changes

```bash
cd /sandbox/payrollmngmt
git pull origin main
```

### Step 2: Clean and Build

```bash
mvn clean package
```

### Step 3: Verify the Tomcat Version

```bash
mvn dependency:tree | grep tomcat-embed-core
```

You should see:
```
org.apache.tomcat.embed:tomcat-embed-core:jar:9.0.109
```

### Step 4: Run Tests

```bash
mvn test
```

Ensure all tests pass.

### Step 5: Start the Application

**Option A - Using Maven:**
```bash
mvn spring-boot:run
```

**Option B - Using the JAR:**
```bash
java -jar target/payrollmngmt-0.0.1-SNAPSHOT.jar
```

**Option C - Using Docker (if applicable):**
```bash
docker-compose up --build
```

### Step 6: Verify Application Health

Test the application endpoints:

```bash
# Home page
curl http://localhost:8080/

# Employees list
curl http://localhost:8080/employees

# Health check (if enabled)
curl http://localhost:8080/actuator/health
```

## Rollback Plan (If Needed)

If issues arise, you can temporarily rollback:

1. **Remove the tomcat.version property** from `pom.xml`:
   ```bash
   git revert <commit-hash>
   ```

2. **Rebuild and redeploy** with the previous version

⚠️ **Warning**: Rollback should only be temporary. The vulnerability must be addressed.

## FAQ

### Q: Do I need to change any application code?
**A**: No, this is purely a dependency upgrade. No code changes are required.

### Q: Will this affect application performance?
**A**: No significant performance impact is expected. Tomcat 9.0.109 is a patch release focused on security fixes.

### Q: Is this compatible with Spring Boot 2.7.18?
**A**: Yes, Tomcat 9.0.109 is fully compatible with Spring Boot 2.7.18.

### Q: What if I'm using Tomcat 10.x or 11.x?
**A**: This project uses Spring Boot 2.7.18, which is designed for Tomcat 9.x. If you've customized to use Tomcat 10.x or 11.x, update to:
- Tomcat 10.x: Use version **10.1.45** or higher
- Tomcat 11.x: Use version **11.0.11** or higher

### Q: Do I need to update any configuration files?
**A**: No, all configuration files (application.properties, application.yml) remain unchanged.

### Q: What about Docker deployments?
**A**: The Docker image will automatically use the updated Tomcat version when rebuilt. Just run:
```bash
docker-compose down
docker-compose build --no-cache
docker-compose up
```

## Troubleshooting

### Issue: Build fails with dependency resolution error

**Solution**: Clean the local Maven cache and retry:
```bash
rm -rf ~/.m2/repository/org/apache/tomcat/embed/
mvn clean package
```

### Issue: Application fails to start

**Solution**: 
1. Check Java version: Requires Java 21
   ```bash
   java -version
   ```
2. Check for port conflicts: Ensure port 8080 is available
   ```bash
   lsof -i :8080
   ```

### Issue: Tests are failing

**Solution**: 
1. Review test logs: `mvn test -X`
2. Check if tests are environment-specific
3. Ensure H2 database is properly initialized

## Additional Resources

- Full remediation report: See `CVE-2025-55752-REMEDIATION.md`
- Original pom.xml: Available in git history
- Apache Tomcat 9.0.109 Release Notes: https://tomcat.apache.org/

## Support

For questions or issues:
1. Check the remediation report: `CVE-2025-55752-REMEDIATION.md`
2. Review application logs: `logs/spring.log`
3. Contact the development team
4. Open a ticket in the project management system

---

**Last Updated**: $(date)  
**Related CVE**: CVE-2025-55752  
**Severity**: HIGH  
**Status**: ✅ Remediated
