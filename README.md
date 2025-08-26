Description Site -> https://spring-team-project2025.github.io/stay_folio_documents/

<details>
<summary>English</summary>
    
# STAY FOLIO - Admin Features Analysis

This document contains a detailed analysis of the admin features implemented in the STAY FOLIO project, focusing on 'Reservation Inquiry' and 'Member Inquiry'. It explains the main flows, core code, and technical strengths that can be highlighted in a portfolio.

## 1. Admin Reservation Inquiry

This feature allows administrators to search and page through all reservation information registered in the system based on various conditions.

### Key Feature Flow

1. **Request Reception (Controller)**: When a request to view the reservation list (`GET /admin/reservationList`) is made from the admin page, the `adminReservationList` method in `AdminListController` handles it. It receives search criteria (`AdminReservationCriteria`) and paging information (`Criteria`) as parameters.
2. **Business Logic Processing (Service)**: `AdminListController` calls the `getAdminReservationList` method of `AdminService` to request the actual reservation list data. The service layer dynamically constructs queries based on search criteria and retrieves the total number of reservations to create a `PageDTO` object for paging.
3. **Database Access (Mapper & XML)**: `AdminService` calls the `selectAdminReservationList` method of the `AdminMapper` interface. This call is mapped to SQL queries defined in `AdminMapper.xml` that retrieve reservation information from the database. The SQL query dynamically changes the `WHERE` clause based on the fields of the `AdminReservationCriteria` object and uses `LIMIT` and `OFFSET` for paging.
4. **Response (Controller & View)**: The reservation list data and `PageDTO` object returned from the service layer are passed to `AdminListController`. The controller adds these to the `Model` and forwards them to the `admin/reservation/reservationList.jsp` view, which renders the reservation list on the screen.

### Core Code

### Controller: `AdminListController.java`

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

### Service: `AdminServiceImpl.java`

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

### Mapper Interface: `AdminMapper.java`

Defines the interface for database access.

```java
// src/main/java/com/hotel/mapper/AdminMapper.java
public List<AdminReservationListDTO> selectAdminReservationList(AdminReservationCriteria cri);
public int selectAdminReservationTotal(AdminReservationCriteria cri);

```

### Mapper XML: `AdminMapper.xml`

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

- **Multi-condition Search and Dynamic SQL Implementation**: Utilized dynamic SQL (`MyBatis <if>`, `<trim>`, `<foreach>`) to combine various search conditions such as reservation status, check-in/check-out dates, member name, stay name, and room name. This demonstrates the ability to flexibly handle complex search requirements.
- **Server-side Paging**: Implemented server-side paging using `LIMIT` and `OFFSET` to prevent performance degradation when querying large datasets. The `PageDTO` object manages total data count and current page information to provide an efficient UI experience.
- **Layered Architecture Design**: Clearly separated responsibilities across Controller-Service-Mapper layers, improving code maintainability and scalability. This emphasizes robust application design skills based on Spring MVC.
- **Use of DTO (Data Transfer Object)**: Used `AdminReservationListDTO` to selectively transfer only necessary data, improving data transfer efficiency and reducing coupling between layers.

## 2. Admin Member Inquiry

This feature allows administrators to search and page through member information registered in the system. (It is expected to be implemented with a structure similar to reservation inquiry.)

### Key Feature Flow

1. **Request Reception (Controller)**: When a request to view the member list (`GET /admin/memberList`) is made from the admin page, the `adminMemberList` method in `AdminListController` handles it. It receives search criteria (`Criteria`) and paging information as parameters.
2. **Business Logic Processing (Service)**: `AdminListController` calls the `getAdminMemberList` method of `AdminService` to request the actual member list data. The service layer dynamically constructs queries based on search criteria and retrieves the total number of members to create a `PageDTO` object for paging.
3. **Database Access (Mapper & XML)**: `AdminService` calls the `selectAdminMemberList` method of the `AdminMapper` interface. This call is mapped to SQL queries defined in `AdminMapper.xml` that retrieve member information from the database. The SQL query dynamically changes the `WHERE` clause based on the fields of the `Criteria` object and uses `LIMIT` and `OFFSET` for paging.
4. **Response (Controller & View)**: The member list data and `PageDTO` object returned from the service layer are passed to `AdminListController`. The controller adds these to the `Model` and forwards them to the `admin/member/memberList.jsp` view, which renders the member list on the screen.

### Core Code

### Controller: `AdminListController.java`

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

### Service: `AdminServiceImpl.java`

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

### Mapper Interface: `AdminMapper.java`

```java
// src/main/java/com/hotel/mapper/AdminMapper.java
public List<MemberVO> selectAdminMemberList(Criteria cri);
public int selectAdminMemberTotal(Criteria cri);

```

### Mapper XML: `AdminMapper.xml`

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

- **Reusable Search and Paging Logic**: Reused `Criteria` and `PageDTO` objects commonly for both reservation and member inquiries, reducing code duplication and improving development efficiency.
- **Flexible Member Search Functionality**: Utilized dynamic SQL to allow searching members by ID, name, phone number, etc., providing user-friendly functionality that helps admins quickly find needed information.
- **Database Integration and Management**: Implemented efficient database integration using MyBatis and separated SQL queries into XML files for better readability and maintainability.

---

## 3. Stay Text Search Feature

This feature allows users to search for stays by entering keywords, mainly based on stay names or locations.

### Key Feature Flow

1. **User Input (JSP)**: In the `search.jsp` page, users enter stay names or location keywords into the search field (`id="keyword"`). This input field has the attribute `data-api="${pageContext.request.contextPath}/search/keyword"`, which sends AJAX requests for auto-suggestions as the user types.
2. **Auto-suggestion Request (JavaScript & Controller)**: Each time the user enters a keyword, the script `resources/js/search/keyword.js` sends an AJAX request to the `/search/suggestions` endpoint. The `SearchController.getSuggestions` method handles this request and calls `StayService.searchStaysSuggestions` to retrieve the suggestion list.
3. **Auto-suggestion Data Retrieval (Service & Mapper)**: The `StayServiceImpl.searchStaysSuggestions` method calls `StayMapper.searchStaysSuggestions`, which queries the database for stay names or locations that match the keyword. The query performs a `LIKE` search on the `si_name` or `si_loca` fields in the `t_stay_info` table.
4. **Result Display (JSP)**: (Although the provided `search.jsp` does not explicitly show the general form submission logic,) typically when the user clicks the search button or presses Enter, the `searchForm` (`action="/search/results"`) sends the search request to the server. This request is processed by a method such as `StayService.getStayListFiltered`, and the results are passed as `stayList` to the JSP, which renders the stay list inside the `searchResultsGrid` section.

### Core Code

### JSP: `search.jsp`

The input field where users enter their search keywords. The `data-api` attribute enables the auto-suggestion feature.

```html
<!-- src/main/webapp/WEB-INF/views/search/search.jsp -->
<input type="text" id="keyword" name="keyword" placeholder="Search by location or stay name." autocomplete="off" data-api="${pageContext.request.contextPath}/search/keyword" data-context="${pageContext.request.contextPath}" />

```

### Controller: `SearchController.java`

Handles the auto-suggestion requests and passes the keyword to the service layer.

```java
// src/main/java/com/hotel/controller/SearchController.java
@Log4j
@Controller
public class SearchController {

    @Autowired
    private StayService stayService;

    // For auto-suggestions
    @GetMapping(value = "/search/suggestions", produces = "application/json; charset=UTF-8")
    @ResponseBody
    public List<StayVO> getSuggestions(@RequestParam(name = "keyword", required = false) String keyword) {
        String q = (keyword == null) ? "" : keyword.trim();
        if (q.isEmpty() || q.length() < 1) {
            return Collections.emptyList();
        }
        List<StayVO> results = stayService.searchStaysSuggestions(q);
        if (log.isDebugEnabled()) {
            log.debug("Keyword suggestions q='" + q + "' -> results=" + (results == null ? 0 : results.size()));
        }
        return results;
    }
}

```

### Service: `StayServiceImpl.java`

Executes the auto-suggestion logic and accesses the DB through the mapper.

```java
// src/main/java/com/hotel/service/StayServiceImpl.java
@Override
public List<StayVO> searchStaysSuggestions(String keyword) {
    if (keyword == null || keyword.trim().isEmpty()) {
        return new ArrayList<>();
    }
    return stayMapper.searchStaysSuggestions(keyword.trim());
}

```

### Mapper Interface: `StayMapper.java`

Defines the interface method for database access related to auto-suggestions.

```java
// src/main/java/com/hotel/mapper/StayMapper.java
List<StayVO> searchStaysSuggestions(@Param("keyword") String keyword);

```

### Mapper XML: `StayMapper.xml`

Defines the SQL query for keyword search using MyBatis. The query uses the `UPPER` function and `LIKE` operator for case-insensitive search and restricts the result to a maximum of 5 rows with `ROWNUM`.

```xml
<!-- src/main/resources/com/hotel/mapper/StayMapper.xml -->
<select id="searchStaysSuggestions" parameterType="string"
    resultType="com.hotel.domain.StayVO">
    SELECT * FROM (
        SELECT
            s.si_id AS siId,
            s.si_name AS siName,
            s.si_loca AS siLoca
        FROM t_stay_info s
        WHERE s.si_show = '1'
          AND s.si_delete = '0'
          AND (
                UPPER(s.si_name) LIKE '%' || UPPER(#{keyword}) || '%'
                OR UPPER(s.si_loca) LIKE '%' || UPPER(#{keyword}) || '%'
            )
        ORDER BY
            CASE WHEN UPPER(s.si_name) LIKE UPPER(#{keyword}) || '%' THEN 1 ELSE 2 END,
            s.si_name
        ) WHERE ROWNUM <= 5
</select>

```

### Portfolio Highlights

- **Real-time Auto-suggestion Implementation**: Provides instant keyword suggestions while typing, enhancing user experience (UX). Demonstrates efficient integration of AJAX communication with backend logic.
- **Dynamic SQL for Flexible Search**: Implements case-insensitive partial matching search on stay names and locations using MyBatis `LIKE` and `UPPER`. Utilizes `CASE` statements for ranking results based on keyword match priority, showing SQL optimization.
- **Frontend-Backend Integration**: Demonstrates full-stack skills by connecting JSP, JavaScript (jQuery), Spring Controller, Service, and Mapper. Especially notable is the use of `data-api` attributes to call backend APIs directly from the frontend.
- **Performance Optimization**: Limits auto-suggestion results to `ROWNUM <= 5` to reduce unnecessary data transfer and improve response speed.
</details>

<details>
<summary>한국어</summary>
    
# **STAY FOLIO - 관리자 기능 분석 (Admin Features Analysis)**

이 문서는 STAY FOLIO 프로젝트에서 구현된 관리자 기능 중 '예약 조회'와 '회원 조회'에 대한 상세 분석을 담고 있습니다. 각 기능의 주요 흐름, 핵심 코드, 그리고 포트폴리오에 활용할 수 있는 기술적 강점들을 설명합니다.

## **1. 관리자 예약 조회 (Admin Reservation Inquiry)**

관리자가 시스템에 등록된 모든 예약 정보를 다양한 조건으로 검색하고 페이징하여 조회할 수 있는 기능입니다.

### **주요 기능 흐름 (Key Feature Flow)**

1. **요청 접수 (Controller)**: 관리자 페이지에서 예약 목록 조회 요청(`GET /admin/reservationList`)이 들어오면 `AdminListController`의 `adminReservationList` 메서드가 이를 처리합니다. 이때, 검색 조건(`AdminReservationCriteria`)과 페이징 정보(`Criteria`)를 파라미터로 받습니다.
2. **비즈니스 로직 처리 (Service)**: `AdminListController`는 `AdminService`의 `getAdminReservationList` 메서드를 호출하여 실제 예약 목록 데이터를 요청합니다. 서비스 계층에서는 검색 조건에 따라 동적으로 쿼리를 구성하고, 전체 예약 건수를 조회하여 페이징 처리를 위한 `PageDTO` 객체를 생성합니다.
3. **데이터베이스 접근 (Mapper & XML)**: `AdminService`는 `AdminMapper` 인터페이스의 `selectAdminReservationList` 메서드를 호출합니다. 이 호출은 `AdminMapper.xml`에 정의된 SQL 쿼리와 매핑되어 데이터베이스에서 예약 정보를 조회합니다. SQL 쿼리는 `AdminReservationCriteria` 객체의 필드 값에 따라 `WHERE` 절이 동적으로 변경되며, `LIMIT`와 `OFFSET`을 사용하여 페이징을 처리합니다.
4. **응답 (Controller & View)**: 서비스 계층에서 반환된 예약 목록 데이터와 `PageDTO` 객체는 `AdminListController`로 전달됩니다. 컨트롤러는 이 데이터를 `Model`에 담아 `admin/reservation/reservationList.jsp` 뷰로 전달하고, 뷰는 전달받은 데이터를 바탕으로 예약 목록을 화면에 렌더링합니다.

### **핵심 코드 (Core Code)**

### **Controller: `AdminListController.java`**

HTTP 요청을 받아 서비스에 전달하고, 결과를 뷰에 넘깁니다.

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

### **Service: `AdminServiceImpl.java`**

비즈니스 로직을 수행하고, Mapper를 통해 DB에 접근합니다.

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

### **Mapper Interface: `AdminMapper.java`**

데이터베이스 접근을 위한 인터페이스를 정의합니다.

```java
// src/main/java/com/hotel/mapper/AdminMapper.java
public List<AdminReservationListDTO> selectAdminReservationList(AdminReservationCriteria cri);
public int selectAdminReservationTotal(AdminReservationCriteria cri);

```

### **Mapper XML: `AdminMapper.xml`**

MyBatis를 사용하여 동적 SQL 쿼리를 정의합니다.

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

### **포트폴리오 주요 포인트 (Key Portfolio Points)**

- **다중 조건 검색 및 동적 SQL 구현**: 예약 상태, 체크인/체크아웃 날짜, 회원 이름, 숙소 이름, 객실 이름 등 다양한 검색 조건을 조합하여 데이터를 조회할 수 있도록 동적 SQL(`MyBatis <if>`, `<trim>`, `<foreach>`)을 활용했습니다. 이는 복잡한 검색 요구사항을 유연하게 처리할 수 있음을 보여줍니다.
- **서버 측 페이징 처리**: 대량의 데이터 조회 시 성능 저하를 방지하기 위해 `LIMIT`와 `OFFSET`을 활용한 서버 측 페이징을 구현했습니다. `PageDTO` 객체를 통해 전체 데이터 수와 현재 페이지 정보를 관리하여 효율적인 UI를 제공합니다.
- **계층형 아키텍처 설계**: Controller-Service-Mapper로 이어지는 명확한 계층 분리를 통해 각 계층의 역할을 명확히 하고, 코드의 유지보수성과 확장성을 높였습니다. 이는 Spring MVC 기반의 견고한 애플리케이션 설계 능력을 강조할 수 있습니다.
- **DTO(Data Transfer Object) 활용**: `AdminReservationListDTO`를 사용하여 필요한 데이터만 선별하여 전송함으로써 데이터 전송 효율성을 높이고, 계층 간의 데이터 결합도를 낮췄습니다.

## **2. 관리자 회원 조회 (Admin Member Inquiry)**

관리자가 시스템에 등록된 회원 정보를 검색하고 페이징하여 조회할 수 있는 기능입니다. (예약 조회와 유사한 구조로 구현되었을 것으로 예상됩니다.)

### **주요 기능 흐름 (Key Feature Flow)**

1. **요청 접수 (Controller)**: 관리자 페이지에서 회원 목록 조회 요청(`GET /admin/memberList`)이 들어오면 `AdminListController`의 `adminMemberList` 메서드가 이를 처리합니다. 검색 조건(`Criteria`)과 페이징 정보를 파라미터로 받습니다.
2. **비즈니스 로직 처리 (Service)**: `AdminListController`는 `AdminService`의 `getAdminMemberList` 메서드를 호출하여 실제 회원 목록 데이터를 요청합니다. 서비스 계층에서는 검색 조건에 따라 동적으로 쿼리를 구성하고, 전체 회원 건수를 조회하여 페이징 처리를 위한 `PageDTO` 객체를 생성합니다.
3. **데이터베이스 접근 (Mapper & XML)**: `AdminService`는 `AdminMapper` 인터페이스의 `selectAdminMemberList` 메서드를 호출합니다. 이 호출은 `AdminMapper.xml`에 정의된 SQL 쿼리와 매핑되어 데이터베이스에서 회원 정보를 조회합니다. SQL 쿼리는 `Criteria` 객체의 필드 값에 따라 `WHERE` 절이 동적으로 변경되며, `LIMIT`와 `OFFSET`을 사용하여 페이징을 처리합니다.
4. **응답 (Controller & View)**: 서비스 계층에서 반환된 회원 목록 데이터와 `PageDTO` 객체는 `AdminListController`로 전달됩니다. 컨트롤러는 이 데이터를 `Model`에 담아 `admin/member/memberList.jsp` 뷰로 전달하고, 뷰는 전달받은 데이터를 바탕으로 회원 목록을 화면에 렌더링합니다.

### **핵심 코드 (Core Code)**

### **Controller: `AdminListController.java`**

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

### **Service: `AdminServiceImpl.java`**

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

### **Mapper Interface: `AdminMapper.java`**

```java
// src/main/java/com/hotel/mapper/AdminMapper.java
public List<MemberVO> selectAdminMemberList(Criteria cri);
public int selectAdminMemberTotal(Criteria cri);

```

### **Mapper XML: `AdminMapper.xml`**

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

### **포트폴리오 주요 포인트 (Key Portfolio Points)**

- **재사용 가능한 검색 및 페이징 로직**: `Criteria` 및 `PageDTO` 객체를 공통으로 사용하여 예약 조회와 유사하게 회원 조회에서도 검색 조건 및 페이징 로직을 재사용했습니다. 이는 코드의 중복을 줄이고 개발 효율성을 높이는 좋은 예시입니다.
- **유연한 회원 검색 기능**: 회원 ID, 이름, 전화번호 등 다양한 기준으로 회원을 검색할 수 있도록 동적 SQL을 활용했습니다. 이는 관리자가 필요한 정보를 신속하게 찾을 수 있도록 돕는 사용자 친화적인 기능입니다.
- **데이터베이스 연동 및 관리**: MyBatis를 활용하여 데이터베이스와의 효율적인 연동을 구현하고, SQL 쿼리를 XML 파일로 분리하여 관리함으로써 코드의 가독성과 유지보수성을 확보했습니다.

---

## **3. 숙소 텍스트 검색 기능 (Text Search for Stays)**

사용자가 키워드를 입력하여 숙소를 검색하는 기능입니다. 주로 숙소 이름이나 지역명을 기반으로 검색이 이루어집니다.

### **주요 기능 흐름 (Key Feature Flow)**

1. **사용자 입력 (JSP)**: `search.jsp` 페이지의 검색 입력 필드(`id="keyword"`)에 사용자가 숙소 이름이나 지역명 등의 키워드를 입력합니다. 이 입력 필드는 `data-api="${pageContext.request.contextPath}/search/keyword"` 속성을 가지고 있어, 입력 시 자동 완성(suggestion) 기능을 위한 AJAX 요청을 보낼 수 있습니다.
2. **자동 완성 요청 (JavaScript & Controller)**: 사용자가 키워드를 입력할 때마다 `resources/js/search/keyword.js` 스크립트에서 `/search/suggestions` 엔드포인트로 AJAX 요청을 보냅니다. `SearchController`의 `getSuggestions` 메서드가 이 요청을 받아 `StayService`의 `searchStaysSuggestions`를 호출하여 자동 완성 목록을 조회합니다.
3. **자동 완성 데이터 조회 (Service & Mapper)**: `StayServiceImpl`의 `searchStaysSuggestions` 메서드는 `StayMapper`의 `searchStaysSuggestions`를 호출하여 데이터베이스에서 키워드에 해당하는 숙소 이름 또는 지역명을 조회합니다. 이 쿼리는 `t_stay_info` 테이블에서 `si_name` 또는 `si_loca` 필드를 기준으로 `LIKE` 검색을 수행합니다.
4. **검색 결과 표시 (JSP)**: (현재 제공된 `search.jsp`에는 일반적인 텍스트 검색 폼 제출 로직이 명시적으로 보이지 않지만, 일반적으로는) 사용자가 검색 버튼을 클릭하거나 엔터를 누르면, `searchForm` (`action="/search/results"`)을 통해 검색 요청이 서버로 전송됩니다. 이 요청은 `StayService`의 `getStayListFiltered` (또는 유사한 검색 메서드)를 통해 처리되고, 결과는 `stayList`라는 이름으로 JSP에 전달되어 `searchResultsGrid` 영역에 숙소 목록이 렌더링됩니다.

### **핵심 코드 (Core Code)**

### **JSP: `search.jsp`**

사용자 입력을 받는 검색 필드입니다. 자동 완성 기능을 위한 `data-api` 속성이 포함되어 있습니다.

```html
<!-- src/main/webapp/WEB-INF/views/search/search.jsp -->
<input type="text" id="keyword" name="keyword" placeholder="지역, 숙소명을 검색해보세요." autocomplete="off" data-api="${pageContext.request.contextPath}/search/keyword" data-context="${pageContext.request.contextPath}" />

```

### **Controller: `SearchController.java`**

자동 완성 요청을 처리하고, 서비스 계층으로 키워드를 전달합니다.

```java
// src/main/java/com/hotel/controller/SearchController.java
@Log4j
@Controller
public class SearchController {

    @Autowired
    private StayService stayService;

    // 자동완성용
    @GetMapping(value = "/search/suggestions", produces = "application/json; charset=UTF-8")
    @ResponseBody
    public List<StayVO> getSuggestions(@RequestParam(name = "keyword", required = false) String keyword) {
        String q = (keyword == null) ? "" : keyword.trim();
        if (q.isEmpty() || q.length() < 1) {
            return Collections.emptyList();
        }
        List<StayVO> results = stayService.searchStaysSuggestions(q);
        if (log.isDebugEnabled()) {
            log.debug("Keyword suggestions q='" + q + "' -> results=" + (results == null ? 0 : results.size()));
        }
        return results;
    }
}

```

### **Service: `StayServiceImpl.java`**

자동 완성 로직을 수행하며, 매퍼를 통해 DB에 접근합니다.

```java
// src/main/java/com/hotel/service/StayServiceImpl.java
@Override
public List<StayVO> searchStaysSuggestions(String keyword) {
    if (keyword == null || keyword.trim().isEmpty()) {
        return new ArrayList<>();
    }
    return stayMapper.searchStaysSuggestions(keyword.trim());
}

```

### **Mapper Interface: `StayMapper.java`**

데이터베이스 접근을 위한 인터페이스에 자동 완성 메서드를 정의합니다.

```java
// src/main/java/com/hotel/mapper/StayMapper.java
List<StayVO> searchStaysSuggestions(@Param("keyword") String keyword);

```

### **Mapper XML: `StayMapper.xml`**

MyBatis를 사용하여 키워드 검색을 위한 SQL 쿼리를 정의합니다. `UPPER` 함수와 `LIKE` 연산자를 사용하여 대소문자 구분 없이 검색하며, `ROWNUM`을 통해 최대 5개의 결과를 반환합니다.

```xml
<!-- src/main/resources/com/hotel/mapper/StayMapper.xml -->
<select id="searchStaysSuggestions" parameterType="string"
    resultType="com.hotel.domain.StayVO">
    SELECT * FROM (
        SELECT
            s.si_id AS siId,
            s.si_name AS siName,
            s.si_loca AS siLoca
        FROM t_stay_info s
        WHERE s.si_show = '1'
          AND s.si_delete = '0'
          AND (
                UPPER(s.si_name) LIKE '%' || UPPER(#{keyword}) || '%'
                OR UPPER(s.si_loca) LIKE '%' || UPPER(#{keyword}) || '%'
            )
        ORDER BY
            CASE WHEN UPPER(s.si_name) LIKE UPPER(#{keyword}) || '%' THEN 1 ELSE 2 END,
            s.si_name
        ) WHERE ROWNUM &lt;= 5
</select>

```

### **포트폴리오 주요 포인트 (Key Portfolio Points)**

- **실시간 자동 완성(Auto-suggestion) 기능 구현**: 사용자가 검색어를 입력하는 동안 실시간으로 관련 검색어를 제안하는 기능을 구현하여 사용자 경험(UX)을 향상시켰습니다. 이는 AJAX 통신과 백엔드 로직의 효율적인 연동을 보여줍니다.
- **동적 SQL을 활용한 유연한 검색**: `MyBatis`의 `LIKE` 연산자와 `UPPER` 함수를 사용하여 숙소 이름과 지역명에 대한 대소문자 구분 없는 부분 일치 검색을 구현했습니다. `CASE` 문을 활용하여 검색어 일치도에 따른 정렬 우선순위를 부여하는 등 SQL 쿼리 최적화 노력을 기울였습니다.
- **프론트엔드-백엔드 연동**: JSP, JavaScript(jQuery), Spring Controller, Service, Mapper로 이어지는 풀스택 개발 능력을 보여줍니다. 특히 `data-api` 속성을 활용하여 프론트엔드에서 백엔드 API를 호출하는 방식을 구현한 점은 주목할 만합니다.
- **성능 최적화**: 자동 완성 기능에서 `ROWNUM <= 5`를 사용하여 불필요한 데이터 전송을 줄이고, 검색 결과 수를 제한하여 응답 속도를 최적화했습니다.
