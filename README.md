import javax.ejb.Stateless;
import javax.annotation.Resource;
import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

@Stateless
public class LegDataBean {

    // Inject the LEG datasource
    @Resource(lookup = "java:comp/env/jdbc/LEGDataSource")
    private DataSource legDataSource;

    public void fetchAndPrintLegData(String measureType, String measureNumber) {
        Connection legConn = null;
        PreparedStatement legStmt = null;
        ResultSet rs = null;

        try {
            // Get connection from LEG datasource
            legConn = legDataSource.getConnection();

            // SQL query to fetch data from LEG database
            String sql = "SELECT b.BILL_ID, bv.VOTE, bv.ACTIVE_FLAG, bv.URGENCY, bva.AUTHOR " +
                         "FROM BILL_TBL b " +
                         "JOIN BILL_VERSION_TBL bv ON b.BILL_ID = bv.BILL_ID " +
                         "JOIN BILL_VERSION_AUTHORS_TBL bva ON bv.BILL_VERSION_ID = bva.BILL_VERSION_ID " +
                         "WHERE b.MEASURE_TYPE = ? AND b.MEASURE_NUMBER = ? AND bv.ACTIVE_FLAG = 'Y'";

            legStmt = legConn.prepareStatement(sql);
            legStmt.setString(1, measureType);
            legStmt.setString(2, measureNumber);

            rs = legStmt.executeQuery();

            // Process the result set and print the data
            while (rs.next()) {
                int billId = rs.getInt("BILL_ID");
                String vote = rs.getString("VOTE");
                String activeFlag = rs.getString("ACTIVE_FLAG");
                String urgency = rs.getString("URGENCY");
                String author = rs.getString("AUTHOR");

                // Print the fetched data
                System.out.println("BILL_ID: " + billId);
                System.out.println("VOTE: " + vote);
                System.out.println("ACTIVE_FLAG: " + activeFlag);
                System.out.println("URGENCY: " + urgency);
                System.out.println("AUTHOR: " + author);
                System.out.println("-----------------------------");
            }

        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            // Close all resources
            try {
                if (rs != null) rs.close();
                if (legStmt != null) legStmt.close();
                if (legConn != null) legConn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
