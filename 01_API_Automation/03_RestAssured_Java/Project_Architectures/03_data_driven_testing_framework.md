# 03: Data-Driven Testing Infrastructure (Excel)

### 1. 🚀 Tutorial Overview
In enterprise automation, we don't hardcode data or rely solely on random Faker data for complex scenarios. **Data-Driven Testing (DDT)** allows us to run the same test logic with multiple sets of real-world data stored in external files like Excel.

- **Objective:** Integrate Apache POI to read Excel data and feed it into TestNG @DataProviders.
- **Workflow:** Excel Sheet -> ExcelUtility -> DataProvider -> @Test Method.
- **Dependencies:** Apache POI (Common & OOXML).

---

### 2. 📌 Core Theoretical Pillars

#### A. Externalization of Data
By moving test data to Excel, non-technical stakeholders (Manual Testers, BAs) can update test cases without touching the code. This decouples "What to test" from "How to test."

#### B. The Role of @DataProvider
TestNG's `@DataProvider` is a specialized method that returns a 2D array (`Object[][]`). It acts as a bridge between the data source (Excel) and the test method. The test method will execute once for every row in the data provider.

#### C. Apache POI Library
Standard Java doesn't understand `.xlsx` files. Apache POI is the industry-standard API for manipulating Microsoft Office documents.

---

### 3. 🛠️ Step-by-Step Implementation

#### Phase 1: Dependency Configuration
Update the `pom.xml` to include Apache POI.

```xml
<!-- pom.xml -->
<dependencies>
    <!-- Apache POI for Excel handling -->
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi</artifactId>
        <version>5.2.3</version>
    </dependency>
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi-ooxml</artifactId>
        <version>5.2.3</version>
    </dependency>
</dependencies>
```

#### Phase 2: The Excel Utility Class
Create a reusable utility to read data from any sheet.

```java
// api.utils.ExcelUtility.java
public class ExcelUtility {
    public FileInputStream fi;
    public XSSFWorkbook workbook;
    public XSSFSheet sheet;
    public XSSFRow row;
    public XSSFCell cell;
    String path;

    public ExcelUtility(String path) {
        this.path = path;
    }

    public int getRowCount(String sheetName) throws IOException {
        fi = new FileInputStream(path);
        workbook = new XSSFWorkbook(fi);
        sheet = workbook.getSheet(sheetName);
        int rowcount = sheet.getLastRowNum();
        workbook.close();
        fi.close();
        return rowcount;
    }

    public int getCellCount(String sheetName, int rownum) throws IOException {
        fi = new FileInputStream(path);
        workbook = new XSSFWorkbook(fi);
        sheet = workbook.getSheet(sheetName);
        row = sheet.getRow(rownum);
        int cellcount = row.getLastCellNum();
        workbook.close();
        fi.close();
        return cellcount;
    }

    public String getCellData(String sheetName, int rownum, int colnum) throws IOException {
        fi = new FileInputStream(path);
        workbook = new XSSFWorkbook(fi);
        sheet = workbook.getSheet(sheetName);
        row = sheet.getRow(rownum);
        cell = row.getCell(colnum);

        DataFormatter formatter = new DataFormatter();
        String data;
        try {
            data = formatter.formatCellValue(cell);
        } catch (Exception e) {
            data = "";
        }
        workbook.close();
        fi.close();
        return data;
    }
}
```

#### Phase 3: The DataProviders Class
Create a centralized class for all data providers.

```java
// api.utils.DataProviders.java
public class DataProviders {

    @DataProvider(name="Data")
    public String[][] getAllData() throws IOException {
        String path = System.getProperty("user.dir") + "//testData//UserData.xlsx";
        ExcelUtility xl = new ExcelUtility(path);

        int rownum = xl.getRowCount("Sheet1");
        int colcount = xl.getCellCount("Sheet1", 1);

        String apidata[][] = new String[rownum][colcount];

        for (int i = 1; i <= rownum; i++) {
            for (int j = 0; j < colcount; j++) {
                apidata[i - 1][j] = xl.getCellData("Sheet1", i, j);
            }
        }
        return apidata;
    }

    @DataProvider(name="UserNames")
    public String[] getUserNames() throws IOException {
        String path = System.getProperty("user.dir") + "//testData//UserData.xlsx";
        ExcelUtility xl = new ExcelUtility(path);

        int rownum = xl.getRowCount("Sheet1");
        String apidata[] = new String[rownum];

        for (int i = 1; i <= rownum; i++) {
            apidata[i - 1] = xl.getCellData("Sheet1", i, 1);
        }
        return apidata;
    }
}
```

#### Phase 4: Implementing DDT in Tests
Update `DDTests.java` to consume the data.

```java
// api.tests.DDTests.java
public class DDTests {

    @Test(priority=1, dataProvider="Data", dataProviderClass=DataProviders.class)
    public void testPostUser(String userID, String userName, String fname, String lname, String useremail, String pwd, String ph) {
        User userPayload = new User();
        userPayload.setId(Integer.parseInt(userID));
        userPayload.setUsername(userName);
        userPayload.setFirstName(fname);
        userPayload.setLastName(lname);
        userPayload.setEmail(useremail);
        userPayload.setPassword(pwd);
        userPayload.setPhone(ph);

        Response response = UserEndpoints.createUser(userPayload);
        Assert.assertEquals(response.getStatusCode(), 200);
    }

    @Test(priority=2, dataProvider="UserNames", dataProviderClass=DataProviders.class)
    public void testDeleteUserByName(String userName) {
        Response response = UserEndpoints.deleteUser(userName);
        Assert.assertEquals(response.getStatusCode(), 200);
    }
}
```

---

### 4. 📊 Technical Component Encyclopedia

| Component | Responsibility | Key Method |
| :--- | :--- | :--- |
| **ExcelUtility** | Low-level POI operations (Read/Write). | `getCellData()` |
| **DataProviders** | Bridging Excel data to TestNG. | `@DataProvider` |
| **DDTests** | Executing logic for each row of data. | `dataProvider="Data"` |

---

### 5. 🏭 Real-World Industrial Patterns

#### A. Multiple DataProviders per Sheet
You can create one large Excel sheet but have multiple DataProviders reading different columns (e.g., one for full user creation, one just for usernames for deletion).

#### B. The "Header Mapping" Strategy
In advanced frameworks, instead of using column indices (`j=0`, `j=1`), we use a `Map<String, String>` where the key is the column header. This makes the code resilient to column reordering in Excel.

---

### 🛡️ Content Preservation Note
- Ensure the Excel file is placed in `testData/UserData.xlsx` relative to the project root.
- The `dataProviderClass` attribute is mandatory if the provider is in a different package.
