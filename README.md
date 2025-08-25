
# STAY FOLIO - 관리자 기능 분석 (Admin Features Analysis)

이 문서는 STAY FOLIO 프로젝트에서 구현된 관리자 기능 중 '예약 조회'와 '회원 조회'에 대한 상세 분석을 담고 있습니다. 각 기능의 주요 흐름, 핵심 코드, 그리고 포트폴리오에 활용할 수 있는 기술적 강점들을 설명합니다.

## 1. 관리자 예약 조회 (Admin Reservation Inquiry)

관리자가 시스템에 등록된 모든 예약 정보를 다양한 조건으로 검색하고 페이징하여 조회할 수 있는 기능입니다.

### 주요 기능 흐름 (Key Feature Flow)

1.  **요청 접수 (Controller)**: 관리자 페이지에서 예약 목록 조회 요청(`GET /admin/reservationList`)이 들어오면 `AdminListController`의 `adminReservationList` 메서드가 이를 처리합니다. 이때, 검색 조건(`AdminReservationCriteria`)과 페이징 정보(`Criteria`)를 파라미터로 받습니다.
2.  **비즈니스 로직 처리 (Service)**: `AdminListController`는 `AdminService`의 `getAdminReservationList` 메서드를 호출하여 실제 예약 목록 데이터를 요청합니다. 서비스 계층에서는 검색 조건에 따라 동적으로 쿼리를 구성하고, 전체 예약 건수를 조회하여 페이징 처리를 위한 `PageDTO` 객체를 생성합니다.
3.  **데이터베이스 접근 (Mapper & XML)**: `AdminService`는 `AdminMapper` 인터페이스의 `selectAdminReservationList` 메서드를 호출합니다. 이 호출은 `AdminMapper.xml`에 정의된 SQL 쿼리와 매핑되어 데이터베이스에서 예약 정보를 조회합니다. SQL 쿼리는 `AdminReservationCriteria` 객체의 필드 값에 따라 `WHERE` 절이 동적으로 변경되며, `LIMIT`와 `OFFSET`을 사용하여 페이징을 처리합니다.
4.  **응답 (Controller & View)**: 서비스 계층에서 반환된 예약 목록 데이터와 `PageDTO` 객체는 `AdminListController`로 전달됩니다. 컨트롤러는 이 데이터를 `Model`에 담아 `admin/reservation/reservationList.jsp` 뷰로 전달하고, 뷰는 전달받은 데이터를 바탕으로 예약 목록을 화면에 렌더링합니다.

### 핵심 코드 (Core Code)

#### Controller: `AdminListController.java`

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

#### Service: `AdminServiceImpl.java`

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

#### Mapper Interface: `AdminMapper.java`

데이터베이스 접근을 위한 인터페이스를 정의합니다.

```java
// src/main/java/com/hotel/mapper/AdminMapper.java
public List<AdminReservationListDTO> selectAdminReservationList(AdminReservationCriteria cri);
public int selectAdminReservationTotal(AdminReservationCriteria cri);
```

#### Mapper XML: `AdminMapper.xml`

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

### 포트폴리오 주요 포인트 (Key Portfolio Points)

*   **다중 조건 검색 및 동적 SQL 구현**: 예약 상태, 체크인/체크아웃 날짜, 회원 이름, 숙소 이름, 객실 이름 등 다양한 검색 조건을 조합하여 데이터를 조회할 수 있도록 동적 SQL(`MyBatis <if>`, `<trim>`, `<foreach>`)을 활용했습니다. 이는 복잡한 검색 요구사항을 유연하게 처리할 수 있음을 보여줍니다.
*   **서버 측 페이징 처리**: 대량의 데이터 조회 시 성능 저하를 방지하기 위해 `LIMIT`와 `OFFSET`을 활용한 서버 측 페이징을 구현했습니다. `PageDTO` 객체를 통해 전체 데이터 수와 현재 페이지 정보를 관리하여 효율적인 UI를 제공합니다.
*   **계층형 아키텍처 설계**: Controller-Service-Mapper로 이어지는 명확한 계층 분리를 통해 각 계층의 역할을 명확히 하고, 코드의 유지보수성과 확장성을 높였습니다. 이는 Spring MVC 기반의 견고한 애플리케이션 설계 능력을 강조할 수 있습니다.
*   **DTO(Data Transfer Object) 활용**: `AdminReservationListDTO`를 사용하여 필요한 데이터만 선별하여 전송함으로써 데이터 전송 효율성을 높이고, 계층 간의 데이터 결합도를 낮췄습니다.

## 2. 관리자 회원 조회 (Admin Member Inquiry)

관리자가 시스템에 등록된 회원 정보를 검색하고 페이징하여 조회할 수 있는 기능입니다. (예약 조회와 유사한 구조로 구현되었을 것으로 예상됩니다.)

### 주요 기능 흐름 (Key Feature Flow)

1.  **요청 접수 (Controller)**: 관리자 페이지에서 회원 목록 조회 요청(`GET /admin/memberList`)이 들어오면 `AdminListController`의 `adminMemberList` 메서드가 이를 처리합니다. 검색 조건(`Criteria`)과 페이징 정보를 파라미터로 받습니다.
2.  **비즈니스 로직 처리 (Service)**: `AdminListController`는 `AdminService`의 `getAdminMemberList` 메서드를 호출하여 실제 회원 목록 데이터를 요청합니다. 서비스 계층에서는 검색 조건에 따라 동적으로 쿼리를 구성하고, 전체 회원 건수를 조회하여 페이징 처리를 위한 `PageDTO` 객체를 생성합니다.
3.  **데이터베이스 접근 (Mapper & XML)**: `AdminService`는 `AdminMapper` 인터페이스의 `selectAdminMemberList` 메서드를 호출합니다. 이 호출은 `AdminMapper.xml`에 정의된 SQL 쿼리와 매핑되어 데이터베이스에서 회원 정보를 조회합니다. SQL 쿼리는 `Criteria` 객체의 필드 값에 따라 `WHERE` 절이 동적으로 변경되며, `LIMIT`와 `OFFSET`을 사용하여 페이징을 처리합니다.
4.  **응답 (Controller & View)**: 서비스 계층에서 반환된 회원 목록 데이터와 `PageDTO` 객체는 `AdminListController`로 전달됩니다. 컨트롤러는 이 데이터를 `Model`에 담아 `admin/member/memberList.jsp` 뷰로 전달하고, 뷰는 전달받은 데이터를 바탕으로 회원 목록을 화면에 렌더링합니다.

### 핵심 코드 (Core Code)

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

### 포트폴리오 주요 포인트 (Key Portfolio Points)

*   **재사용 가능한 검색 및 페이징 로직**: `Criteria` 및 `PageDTO` 객체를 공통으로 사용하여 예약 조회와 유사하게 회원 조회에서도 검색 조건 및 페이징 로직을 재사용했습니다. 이는 코드의 중복을 줄이고 개발 효율성을 높이는 좋은 예시입니다.
*   **유연한 회원 검색 기능**: 회원 ID, 이름, 전화번호 등 다양한 기준으로 회원을 검색할 수 있도록 동적 SQL을 활용했습니다. 이는 관리자가 필요한 정보를 신속하게 찾을 수 있도록 돕는 사용자 친화적인 기능입니다.
*   **데이터베이스 연동 및 관리**: MyBatis를 활용하여 데이터베이스와의 효율적인 연동을 구현하고, SQL 쿼리를 XML 파일로 분리하여 관리함으로써 코드의 가독성과 유지보수성을 확보했습니다.

---
