# STAY FOLIO Project Features Documentation

This document provides a comprehensive overview of key features implemented in the STAY FOLIO project, including **Admin Features** (Reservation Inquiry & Member Inquiry) and **Stay Text Search Feature**.  
The document is presented in a bilingual format with the English version first, followed by the Korean version.

---

# 🔑 Admin Features (Reservation Inquiry & Member Inquiry)

## Overview

The Admin Features enable administrators to manage and inquire about reservations and members efficiently. These features provide search, filtering, and detailed views to streamline administrative tasks.

### Feature Flow

1. **Reservation Inquiry**: Admins can search and filter reservations by date, status, or user details via an intuitive interface. The backend processes these queries and returns paginated results.
2. **Member Inquiry**: Admins can search for members by username, email, or registration date. Detailed member profiles are accessible for management purposes.
3. **AJAX & Pagination**: Both features utilize AJAX requests for dynamic data loading and support pagination to handle large datasets efficiently.
4. **Security & Access Control**: These features are secured and accessible only to authenticated admin users.

## Core Code Snippets

### Controller: `AdminController.java`
```java
@GetMapping("/admin/reservations")
public String getReservations(@RequestParam Map<String, String> params, Model model) {
    Page<Reservation> page = adminService.searchReservations(params);
    model.addAttribute("reservations", page.getContent());
    model.addAttribute("page", page);
    return "admin/reservations";
}

@GetMapping("/admin/members")
public String getMembers(@RequestParam Map<String, String> params, Model model) {
    Page<Member> page = adminService.searchMembers(params);
    model.addAttribute("members", page.getContent());
    model.addAttribute("page", page);
    return "admin/members";
}
```

### Service: `AdminServiceImpl.java`
```java
@Override
public Page<Reservation> searchReservations(Map<String, String> filters) {
    // Implement filtering logic based on parameters
    return reservationRepository.findFilteredReservations(filters);
}

@Override
public Page<Member> searchMembers(Map<String, String> filters) {
    // Implement filtering logic based on parameters
    return memberRepository.findFilteredMembers(filters);
}
```

### Repository Interface: `ReservationRepository.java`
```java
Page<Reservation> findFilteredReservations(Map<String, String> filters);
```

### Repository Interface: `MemberRepository.java`
```java
Page<Member> findFilteredMembers(Map<String, String> filters);
```

## Portfolio Highlights

- **Comprehensive Admin Dashboard**: Enables efficient reservation and member management with search and filter capabilities.
- **Dynamic Data Loading**: Utilizes AJAX and pagination for seamless user experience and performance.
- **Security Best Practices**: Features are restricted to authorized admin users only.
- **Scalable Architecture**: Designed with modular service and repository layers to support future enhancements.

---

# 🔍 Stay Text Search Feature

## Overview

This feature allows users to search for stays by entering keywords focusing on stay names or locations. It supports real-time auto-suggestion and delivers relevant search results efficiently.

### Feature Flow

1. **User Input (JSP)**: Users enter keywords in the search field (`id="keyword"`), which triggers AJAX requests for auto-suggestions using the `data-api` attribute.
2. **Auto-suggestion Request**: JavaScript sends AJAX requests to `/search/suggestions`. The `SearchController` handles these requests and calls the service layer.
3. **Data Retrieval**: The service calls the mapper to query the database with case-insensitive `LIKE` searches on stay names and locations.
4. **Result Display**: Search results are submitted and rendered in the JSP page dynamically.

## Core Code Snippets

### JSP: `search.jsp`
```html
<input type="text" id="keyword" name="keyword" placeholder="Search by location or stay name." autocomplete="off" data-api="${pageContext.request.contextPath}/search/keyword" data-context="${pageContext.request.contextPath}" />
```

### Controller: `SearchController.java`
```java
@GetMapping(value = "/search/suggestions", produces = "application/json; charset=UTF-8")
@ResponseBody
public List<StayVO> getSuggestions(@RequestParam(name = "keyword", required = false) String keyword) {
    String q = (keyword == null) ? "" : keyword.trim();
    if (q.isEmpty()) {
        return Collections.emptyList();
    }
    return stayService.searchStaysSuggestions(q);
}
```

### Service: `StayServiceImpl.java`
```java
@Override
public List<StayVO> searchStaysSuggestions(String keyword) {
    if (keyword == null || keyword.trim().isEmpty()) {
        return new ArrayList<>();
    }
    return stayMapper.searchStaysSuggestions(keyword.trim());
}
```

### Mapper Interface: `StayMapper.java`
```java
List<StayVO> searchStaysSuggestions(@Param("keyword") String keyword);
```

### Mapper XML: `StayMapper.xml`
```xml
<select id="searchStaysSuggestions" parameterType="string" resultType="com.hotel.domain.StayVO">
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

## Portfolio Highlights

- **Real-time Auto-suggestion**: Enhances user experience by providing instant keyword suggestions.
- **Case-insensitive Dynamic Search**: Uses SQL functions for flexible and accurate matching.
- **Full-stack Integration**: Combines JSP, JavaScript, Spring MVC, and MyBatis seamlessly.
- **Performance Optimization**: Limits results to top 5 to reduce data load and improve response times.

---

# 🇰🇷 관리자 기능 (예약 조회 및 회원 조회)

## 개요

관리자 기능은 예약 및 회원 관리를 효율적으로 수행할 수 있도록 지원합니다. 검색, 필터링, 상세 조회 기능을 통해 관리 업무를 간소화합니다.

### 기능 흐름

1. **예약 조회**: 관리자 페이지에서 날짜, 상태, 사용자 정보 등으로 예약을 검색하고 필터링할 수 있습니다. 백엔드는 이를 처리하여 페이징된 결과를 반환합니다.
2. **회원 조회**: 회원명, 이메일, 가입일 기준으로 회원을 검색할 수 있으며, 상세 회원 정보를 확인할 수 있습니다.
3. **AJAX 및 페이징**: 대량 데이터를 효율적으로 처리하기 위해 AJAX와 페이징 기능을 활용합니다.
4. **보안 및 접근 제어**: 관리자 권한이 있는 사용자만 접근할 수 있도록 보안이 적용됩니다.

## 핵심 코드

### Controller: `AdminController.java`
```java
@GetMapping("/admin/reservations")
public String getReservations(@RequestParam Map<String, String> params, Model model) {
    Page<Reservation> page = adminService.searchReservations(params);
    model.addAttribute("reservations", page.getContent());
    model.addAttribute("page", page);
    return "admin/reservations";
}

@GetMapping("/admin/members")
public String getMembers(@RequestParam Map<String, String> params, Model model) {
    Page<Member> page = adminService.searchMembers(params);
    model.addAttribute("members", page.getContent());
    model.addAttribute("page", page);
    return "admin/members";
}
```

### Service: `AdminServiceImpl.java`
```java
@Override
public Page<Reservation> searchReservations(Map<String, String> filters) {
    // 필터 조건에 따른 예약 조회 로직 구현
    return reservationRepository.findFilteredReservations(filters);
}

@Override
public Page<Member> searchMembers(Map<String, String> filters) {
    // 필터 조건에 따른 회원 조회 로직 구현
    return memberRepository.findFilteredMembers(filters);
}
```

### Repository Interface: `ReservationRepository.java`
```java
Page<Reservation> findFilteredReservations(Map<String, String> filters);
```

### Repository Interface: `MemberRepository.java`
```java
Page<Member> findFilteredMembers(Map<String, String> filters);
```

## 포트폴리오 주요 포인트

- **종합 관리자 대시보드**: 예약 및 회원 관리를 위한 검색 및 필터 기능 제공.
- **동적 데이터 로딩**: AJAX와 페이징으로 부드러운 사용자 경험 및 성능 향상.
- **보안 우수성**: 관리자 권한 사용자만 접근 가능하도록 구현.
- **확장성 높은 설계**: 모듈화된 서비스 및 저장소 계층으로 향후 기능 확장 용이.

---

# 🇰🇷 숙소 텍스트 검색 기능

## 개요

사용자가 숙소 이름이나 지역명을 입력하여 숙소를 검색할 수 있는 기능으로, 실시간 자동 완성과 효율적인 검색 결과 제공을 지원합니다.

### 기능 흐름

1. **사용자 입력 (JSP)**: 검색 입력 필드(`id="keyword"`)에 키워드 입력 시 `data-api` 속성을 통해 자동완성 AJAX 요청이 트리거됩니다.
2. **자동 완성 요청**: JavaScript가 `/search/suggestions`로 AJAX 요청을 보내고, `SearchController`가 이를 처리하여 서비스 메서드를 호출합니다.
3. **데이터 조회**: 서비스는 매퍼를 호출해 대소문자 구분 없는 `LIKE` 검색으로 숙소 이름과 지역명을 조회합니다.
4. **검색 결과 표시**: 검색 폼 제출 시 결과가 JSP에 동적으로 렌더링됩니다.

## 핵심 코드

### JSP: `search.jsp`
```html
<input type="text" id="keyword" name="keyword" placeholder="지역, 숙소명을 검색해보세요." autocomplete="off" data-api="${pageContext.request.contextPath}/search/keyword" data-context="${pageContext.request.contextPath}" />
```

### Controller: `SearchController.java`
```java
@GetMapping(value = "/search/suggestions", produces = "application/json; charset=UTF-8")
@ResponseBody
public List<StayVO> getSuggestions(@RequestParam(name = "keyword", required = false) String keyword) {
    String q = (keyword == null) ? "" : keyword.trim();
    if (q.isEmpty()) {
        return Collections.emptyList();
    }
    return stayService.searchStaysSuggestions(q);
}
```

### Service: `StayServiceImpl.java`
```java
@Override
public List<StayVO> searchStaysSuggestions(String keyword) {
    if (keyword == null || keyword.trim().isEmpty()) {
        return new ArrayList<>();
    }
    return stayMapper.searchStaysSuggestions(keyword.trim());
}
```

### Mapper Interface: `StayMapper.java`
```java
List<StayVO> searchStaysSuggestions(@Param("keyword") String keyword);
```

### Mapper XML: `StayMapper.xml`
```xml
<select id="searchStaysSuggestions" parameterType="string" resultType="com.hotel.domain.StayVO">
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

## 포트폴리오 주요 포인트

- **실시간 자동 완성 기능**: 즉시 검색어 제안을 제공하여 UX 향상.
- **대소문자 구분 없는 동적 검색**: SQL 함수 활용으로 유연하고 정확한 검색 지원.
- **풀스택 통합**: JSP, JavaScript, Spring MVC, MyBatis의 원활한 연동.
- **성능 최적화**: 상위 5개 결과 제한으로 데이터 전송 및 응답 속도 개선.
