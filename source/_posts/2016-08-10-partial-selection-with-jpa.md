---
layout: post
title: Partial selection of Spring Data JPA
date: 2016-08-10
tags:
- Spring
- Spring Data JPA
- ORM
- Hibernate
categories: Spring
description: Partial selection of Spring Data JPA
---

When we use `select` to retrive some data from MySQL, we do not want to get the whole row with all columns. 
For example, I have the following `tab_staff_info` table with nearly 100 columns:
```java
@Entity
@Table(name = "tab_staff_info")
@DynamicUpdate
public class StaffInfo extends BaseEntity {
    /**
     * qq号
     */
    @Column(length = 40)
    private String qqNo;

    /**
     * 公司Id
     */
    @Column(length = 40)
    private String companyId;
    /**
     * 工号
     */
    @Column(length = 32)
    private String staffNo;
    /**
     * 姓名
     */
    @Column(length = 32)
    private String staffName;
    /**
     * 证件号
     */
    @Column(length = 30)
    private String idCardNo;
    /**
     * 证件类型
     */
    @Column(length = 20)
    @Enumerated(EnumType.STRING)
    private IdCardType idCardType;
    /**
     * 生日
     */
    @Column
    private Date birthday;
    /**
     * 性别
     */
    @Column(length = 10)
    @Enumerated(EnumType.STRING)
    private SexType sex;
    /**
     * 手机号码
     */
    @Column(length = 18)
    private String mobileNo;

    // A lot more columns..
}
```
In most cases, I only want to get some (not all) of the columns. For example, I want to get the basic information, which are: `companyId`, `staffNo`, `staffName` and `idCardNo`.

In order to do this, I need to modify the `StaffInfoDao` interface.

```java
public interface StaffInfoDao extends JpaRepository<StaffInfo, String> {
      @Query("select new StaffInfo(s.companyId, s.staffNo, s.staffName, s.idCardNo) from StaffInfo s where s.companyId = :companyId")
      List<StaffInfo> findStaffBasicInfoByCompanyId(@Param("companyId") String companyId);
}
```
And of couse we need to add a constructor to support `new StaffInfo(s.companyId, s.staffNo, s.staffName, s.idCardNo)` in the JPA QL:
```java
public StaffInfo(String companyId, String staffNo, String staffName, String idCardNo) {
      this.companyId = companyId;
      this.staffNo = staffNo;
      this.staffName = staffName;
      this.idCardNo = idCardNo;
}
```
Change log level of hibernate library to debug and we can see the following output:
```
[20160808 14:30:14][DEBUG][org.hibernate.SQL](SqlStatementLogger.java:109) - select staffinfo0_.companyId as col_0_0_, staffinfo0_.staffNo as col_1_0_, staffinfo0_.staffName as col_2_0_, staffinfo0_.idCardNo as col_3_0_ from tab_staff_info staffinfo0_ where staffinfo0_.companyId=?
[20160808 14:30:14][DEBUG][org.hibernate.engine.jdbc.internal.LogicalConnectionImpl](LogicalConnectionImpl.java:226) - Obtaining JDBC connection
[20160808 14:30:14][DEBUG][org.hibernate.engine.jdbc.internal.LogicalConnectionImpl](LogicalConnectionImpl.java:232) - Obtained JDBC connection
```

