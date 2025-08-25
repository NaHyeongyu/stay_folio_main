# STAY FOLIO - Admin Features Analysis

This document contains a detailed analysis of the admin features implemented in the STAY FOLIO project, focusing on 'Reservation Inquiry' and 'Member Inquiry'. It explains the main flows, core code, and technical strengths that can be highlighted in a portfolio.

## 1. Admin Reservation Inquiry

This feature allows administrators to search and page through all reservation information registered in the system based on various conditions.

### Key Feature Flow

1.  **Request Reception (Controller)**: When a request to view the reservation list (`GET /admin/reservationList`) is made from the admin page, the `adminReservationList` method in `AdminListController` handles it. It receives search criteria (`AdminReservationCriteria`) and paging information (`Criteria`) as parameters.
2.  **Business Logic Processing (Service)**: `AdminListController` calls the `getAdminReservationList` method of `AdminService` to request the actual reservation list data. The service layer dynamically constructs queries based on search criteria and retrieves the total number of reservations to create a `PageDTO` object for paging.
3.  **Database Access (Mapper & XML)**: `AdminService` calls the `selectAdminReservationList` method of the `AdminMapper` interface. This call is mapped to SQL queries defined in `AdminMapper.xml` that retrieve reservation information from the database. The SQL query dynamically changes the `WHERE` clause based on the fields of the `AdminReservationCriteria` object and uses `LIMIT` and `OFFSET` for paging.
4.  **Response (Controller & View)**: The reservation list data and `PageDTO` object returned from the service layer are passed to `AdminListController`. The controller adds these to the `Model` and forwards them to the `admin/reservation/reservationList.jsp` view, which renders the reservation list on the screen.

### Core Code

#### Controller: `AdminListController.java`

Handles HTTP requests, passes them to the service, and forwards results to the view.

```java
// src/main/java/com/hotel/controller/AdminListController.java
@GetMapping("/reservationList")
public void adminReservationList(AdminReservationCriteria cri, Model model) {
    log.info("adminReservationList: " + cri);
    model.addAttribute("list", adminService.getAdminReservationList(cri));
    int total = adminService.getAdminReservationTotal(cri);
    log.info("total: " + total);
    model.addAttribute("pageMaker", new PageDTO(cri, total));
}
```

#### Service: `AdminServiceImpl.java`

Performs business logic and accesses the database via the Mapper.

```java
// src/main/java/com/hotel/service/AdminServiceImpl.java
@Override
public List<AdminReservationListDTO> getAdminReservationList(AdminReservationCriteria cri) {
    log.info("getAdminReservationList: " + cri);
    return adminMapper.selectAdminReservationList(cri);
}

@Override
public int getAdminReservationTotal(AdminReservationCriteria cri) {
    log.info("getAdminReservationTotal: " + cri);
    return adminMapper.selectAdminReservationTotal(cri);
}
```

#### Mapper Interface: `AdminMapper.java`

Defines the interface for database access.

```java
// src/main/java/com/hotel/mapper/AdminMapper.java
public List<AdminReservationListDTO> selectAdminReservationList(AdminReservationCriteria cri);
public int selectAdminReservationTotal(AdminReservationCriteria cri);
```

#### Mapper XML: `AdminMapper.xml`

Defines dynamic SQL queries using MyBatis.

```xml
<!-- src/main/resources/com/hotel/mapper/AdminMapper.xml -->
<select id="selectAdminReservationList" resultType="com.hotel.domain.AdminReservationListDTO">
    SELECT
        r.r_num, r.r_checkin, r.r_checkout, r.r_price, r.r_status,
        m.m_id, m.m_name,
        s.s_name,
        ro.ro_name
    FROM
        reservation r
    JOIN
        member m ON r.m_num = m.m_num
    JOIN
        room ro ON r.ro_num = ro.ro_num
    JOIN
        stay s ON ro.s_num = s.s_num
    <include refid="criteria"></include>
    ORDER BY r.r_num DESC
    LIMIT #{amount} OFFSET #{skip}
</select>

<select id="selectAdminReservationTotal" resultType="int">
    SELECT count(*) FROM reservation r
    JOIN member m ON r.m_num = m.m_num
    JOIN room ro ON r.ro_num = ro.ro_num
    JOIN stay s ON ro.s_num = s.s_num
    <include refid="criteria"></include>
</select>

<sql id="criteria">
    <where>
        <if test="type != null and keyword != null">
            <trim prefix="(" suffix=")" prefixOverrides="OR">
                <foreach item="item" collection="typeArr">
                    <if test="item == 'M'.toString()">
                        OR m.m_name LIKE CONCAT('%', #{keyword}, '%')
                    </if>
                    <if test="item == 'S'.toString()">
                        OR s.s_name LIKE CONCAT('%', #{keyword}, '%')
                    </if>
                    <if test="item == 'R'.toString()">
                        OR ro.ro_name LIKE CONCAT('%', #{keyword}, '%')
                    </if>
                </foreach>
            </trim>
        </if>
        <if test="r_status != null and r_status != ''">
            AND r.r_status = #{r_status}
        </if>
        <if test="checkinDate != null and checkinDate != ''">
            AND r.r_checkin &gt;= #{checkinDate}
        </if>
        <if test="checkoutDate != null and checkoutDate != ''">
            AND r.r_checkout &lt;= #{checkoutDate}
        </if>
    </where>
</sql>
```

### Key Portfolio Points

*   **Multi-condition Search and Dynamic SQL Implementation**: Utilized dynamic SQL (`MyBatis <if>`, `<trim>`, `<foreach>`) to combine various search conditions such as reservation status, check-in/check-out dates, member name, stay name, and room name. This demonstrates the ability to flexibly handle complex search requirements.
*   **Server-side Paging**: Implemented server-side paging using `LIMIT` and `OFFSET` to prevent performance degradation when querying large datasets. The `PageDTO` object manages total data count and current page information to provide an efficient UI experience.
*   **Layered Architecture Design**: Clearly separated responsibilities across Controller-Service-Mapper layers, improving code maintainability and scalability. This emphasizes robust application design skills based on Spring MVC.
*   **Use of DTO (Data Transfer Object)**: Used `AdminReservationListDTO` to selectively transfer only necessary data, improving data transfer efficiency and reducing coupling between layers.

## 2. Admin Member Inquiry

This feature allows administrators to search and page through member information registered in the system. (It is expected to be implemented with a structure similar to reservation inquiry.)

### Key Feature Flow

1.  **Request Reception (Controller)**: When a request to view the member list (`GET /admin/memberList`) is made from the admin page, the `adminMemberList` method in `AdminListController` handles it. It receives search criteria (`Criteria`) and paging information as parameters.
2.  **Business Logic Processing (Service)**: `AdminListController` calls the `getAdminMemberList` method of `AdminService` to request the actual member list data. The service layer dynamically constructs queries based on search criteria and retrieves the total number of members to create a `PageDTO` object for paging.
3.  **Database Access (Mapper & XML)**: `AdminService` calls the `selectAdminMemberList` method of the `AdminMapper` interface. This call is mapped to SQL queries defined in `AdminMapper.xml` that retrieve member information from the database. The SQL query dynamically changes the `WHERE` clause based on the fields of the `Criteria` object and uses `LIMIT` and `OFFSET` for paging.
4.  **Response (Controller & View)**: The member list data and `PageDTO` object returned from the service layer are passed to `AdminListController`. The controller adds these to the `Model` and forwards them to the `admin/member/memberList.jsp` view, which renders the member list on the screen.

### Core Code

#### Controller: `AdminListController.java`

```java
// src/main/java/com/hotel/controller/AdminListController.java
@GetMapping("/memberList")
public void adminMemberList(Criteria cri, Model model) {
    log.info("adminMemberList: " + cri);
    model.addAttribute("list", adminService.getAdminMemberList(cri));
    int total = adminService.getAdminMemberTotal(cri);
    log.info("total: " + total);
    model.addAttribute("pageMaker", new PageDTO(cri, total));
}
```

#### Service: `AdminServiceImpl.java`

```java
// src/main/java/com/hotel/service/AdminServiceImpl.java
@Override
public List<MemberVO> getAdminMemberList(Criteria cri) {
    log.info("getAdminMemberList: " + cri);
    return adminMapper.selectAdminMemberList(cri);
}

@Override
public int getAdminMemberTotal(Criteria cri) {
    log.info("getAdminMemberTotal: " + cri);
    return adminMapper.selectAdminMemberTotal(cri);
}
```

#### Mapper Interface: `AdminMapper.java`

```java
// src/main/java/com/hotel/mapper/AdminMapper.java
public List<MemberVO> selectAdminMemberList(Criteria cri);
public int selectAdminMemberTotal(Criteria cri);
```

#### Mapper XML: `AdminMapper.xml`

```xml
<!-- src/main/resources/com/hotel/mapper/AdminMapper.xml -->
<select id="selectAdminMemberList" resultType="com.hotel.domain.MemberVO">
    SELECT * FROM member
    <include refid="memberCriteria"></include>
    ORDER BY m_num DESC
    LIMIT #{amount} OFFSET #{skip}
</select>

<select id="selectAdminMemberTotal" resultType="int">
    SELECT count(*) FROM member
    <include refid="memberCriteria"></include>
</select>

<sql id="memberCriteria">
    <where>
        <if test="type != null and keyword != null">
            <trim prefix="(" suffix=")" prefixOverrides="OR">
                <foreach item="item" collection="typeArr">
                    <if test="item == 'I'.toString()">
                        OR m_id LIKE CONCAT('%', #{keyword}, '%')
                    </if>
                    <if test="item == 'N'.toString()">
                        OR m_name LIKE CONCAT('%', #{keyword}, '%')
                    </if>
                    <if test="item == 'P'.toString()">
                        OR m_phone LIKE CONCAT('%', #{keyword}, '%')
                    </if>
                </foreach>
            </trim>
        </if>
    </where>
</sql>
```

### Key Portfolio Points

*   **Reusable Search and Paging Logic**: Reused `Criteria` and `PageDTO` objects commonly for both reservation and member inquiries, reducing code duplication and improving development efficiency.
*   **Flexible Member Search Functionality**: Utilized dynamic SQL to allow searching members by ID, name, phone number, etc., providing user-friendly functionality that helps admins quickly find needed information.
*   **Database Integration and Management**: Implemented efficient database integration using MyBatis and separated SQL queries into XML files for better readability and maintainability.

---
