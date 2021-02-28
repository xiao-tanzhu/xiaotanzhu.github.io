---
layout: post
title: Spring中使用JUnit+mockito+powermock进行单元测试
date: 2016-07-25
tags:
- Spring
- 单元测试
- JUnit
- Mockito
- PowerMock
categories: Spring
description: Spring中使用JUnit+mockito+powermock进行单元测试
---
Spring中执行单元测试，最麻烦的就是解决Bean的定义以及注入的问题。最开始使用Spring的上下文初始化进行测试，开头是这样的：
```java
@RunWith(SpringJUnit4ClassRunner.class) 
@ContextConfiguration("/config/Spring-db1.xml") 
```
于是，为了能让这个单元测试正常运行起来，我又Mock了一堆其他的如：MySQL，MongoDB，Redis等等无数的组件。最终测试终于可以运行起来，但是运行的时候又需要对整个Spring的上下文进行初始化，跑一个单元测试需要0.1秒，跑初始化流程就需要1分钟。不过当时单元测试并不是团队高优先级的任务，后来也就没有再研究。   

最近回归Bug频频出现，单元测试又开始提上日程。花了大半天时间研究了JUnit+mockito+powermock进行可行的单元测试。


## 三个软件的定位
- **JUnit** 作为优秀的测试框架，在Spring单元测试占有相当大的市场份额
- **Mockito** 管理Spring的Mock对象管理，以及依赖注入等
- **PowerMock** Mockito不能对构造函数、静态函数以及私有函数进行Stunning，PowerMock是Mockito基础上的增强，填补了后者这方面的空白

## 从一个例子开始：签到
凡事从简单的开始，我选择了系统中最复杂模块之一————“签到”的最简单部分进行单元测试。以下是需要进行测试的代码：
```java
@Override
@Transactional
public SigninResult signV3(String staffId, SigninType signType, String wifiName, String wifiMac, Double longitude,
                           Double latitude, Double radius, String locationName, String mobileId, Date signDate, String companyId, boolean isSigninOnlyOnce) {
    this.checkOutSign(signType, companyId, staffId, signDate);//校验是否有相同类型的外出签到在申请中或已经审批通过了
    return actualSignV3(staffId, signType, wifiName, wifiMac, longitude, latitude, radius, locationName, mobileId, signDate, companyId, new Date(), isSigninOnlyOnce, false);
}

```
大家可以忽略乱七八糟的参数，只关注函数的两步：
1. 校验和外出签到关联的条件：checkOutSign
2. 实际执行签到的逻辑：actualSignV3
另外需要注意的是：
1. checkOutSign是私有函数，如果其中不符合签到条件的话会抛出异常
2. actualSignV3是共有函数，在某些版本的接口中可以被其他模块直接调用
由于我们要演示对私有函数的测试，所以`checkOutSign`内的大致流程为：
1. 获取一个外出签到记录，`signinOutRecordDao.findByCompanyIdAndStaffIdAndSignTypeAndSignDateAndApplicationStatusIn(xxx, xxx)`。（JPA实现，函数名比较长，勿喷）
2. 校验外出签到，如果有异常的时候，抛出`IrenshiException`

## 单元测试代码
先从代码开始，然后一步步讲解

```java
package cn.irenshi.biz.attendance.service;

import cn.irenshi.biz.attendance.dao.mysql.SigninOutRecordDao;
import cn.irenshi.biz.attendance.service.impl.SignServiceImpl;
import cn.irenshi.meta.dto.attendance.mysql.SigninOutRecord;
import cn.irenshi.meta.entity.attendance.SigninResult;
import cn.irenshi.meta.exception.IrenshiException;
import cn.irenshi.meta.type.ApplicationStatus;
import cn.irenshi.meta.type.SigninType;
import com.google.common.collect.Lists;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.powermock.core.classloader.annotations.PrepareOnlyThisForTest;
import org.powermock.modules.junit4.PowerMockRunner;
import org.powermock.reflect.Whitebox;

import java.util.Date;

import static org.junit.Assert.assertTrue;
import static org.mockito.Matchers.any;
import static org.mockito.Matchers.eq;
import static org.mockito.Mockito.doReturn;
import static org.mockito.Mockito.*;
import static org.powermock.api.mockito.PowerMockito.doNothing;
import static org.powermock.api.mockito.PowerMockito.spy;
import static org.powermock.api.mockito.PowerMockito.*;

// 1. 使用名称为PowerMockRunner的JUnit模块执行单元测试
@RunWith(PowerMockRunner.class)
public class SignServiceTest {
    // 2. 使用Mockito的@InjectMocks注解将待测试的实现类注入
    @InjectMocks
    private SignServiceImpl signService;
    // 3. 将生成MockDao，并注入到@InjectMocks指定的类中
    @Mock
    private SigninOutRecordDao signinOutRecordDao;

    @Test
    // 4. 对于final类，有private函数及static函数的类等，必须使用此注解，之后才能着Stubbing
    @PrepareOnlyThisForTest(SignServiceImpl.class)
    public void testSignV3() throws Exception {
        String staffId = "mockStaffId";
        SigninType signType = SigninType.SIGNIN_AFTERNOON;
        String wifiName = "mockWifiName";
        String wifiMac = "mockWifiMac";
        Double longitude = 0.0;
        Double latitude = 0.0;
        Double radius = 0.0;
        String locationName = "mockLocationName";
        String mobileId = "mockMobileId";
        Date signDate = new Date();
        String companyId = "mockCompanyId";
        boolean isSigninOnlyOnce = true;

        // 5. 对实体类进行Stubbing，从spy()开始
        SignServiceImpl spy = spy(signService);

        SigninResult signinResult = new SigninResult();

        // 6. 对私有函数进行Stubbing
        doNothing().when(spy, "checkOutSign", signType, companyId, staffId, signDate);
        // 7. 对共有和函数进行Stubbing
        // 8. 因为actualSignV3含有不确定的变量，所以必须使用Matchers进行参数处理
        doReturn(signinResult).when(spy).actualSignV3(eq(staffId), eq(signType), eq(wifiName), eq(wifiMac),
                eq(longitude), eq(latitude), eq(radius), eq(locationName), eq(mobileId), eq(signDate), eq(companyId),
                any(), eq(isSigninOnlyOnce), eq(false));

        // 9. 执行即将进行测试的代码
        SigninResult result = spy.signV3(staffId, signType, wifiName, wifiMac, longitude, latitude, radius, locationName, mobileId,
                signDate, companyId, isSigninOnlyOnce);

        // 10. 检查该私有函数是否以给定的参数被调用了1次
        verifyPrivate(spy, times(1)).invoke("checkOutSign", signType, companyId, staffId, signDate);
        // 11. 检查该共有函数是否以给定的参数被调用了1次
        // 12. 同样由于含有不确定变量，校验的时候也需要使用Matchers对参数进行处理
        verify(spy, times(1)).actualSignV3(eq(staffId), eq(signType), eq(wifiName), eq(wifiMac),
                eq(longitude), eq(latitude), eq(radius), eq(locationName), eq(mobileId), eq(signDate), eq(companyId),
                any(), eq(isSigninOnlyOnce), eq(false));
        // 13. 校验函数的返回值是否正确
        assertTrue(signinResult == result);
    }

    @Test
    public void testCheckOutSign1() throws Exception {
        String staffId = "mockStaffId";
        SigninType signType = SigninType.SIGNIN_AFTERNOON;
        Date signDate = new Date();
        String companyId = "mockCompanyId";

        SigninOutRecord record1 = new SigninOutRecord();
        SigninOutRecord record2 = new SigninOutRecord();
        SigninOutRecord record3 = new SigninOutRecord();
        SigninOutRecord record4 = new SigninOutRecord();
        SigninOutRecord record5 = new SigninOutRecord();
        record1.setApplicationStatus(ApplicationStatus.CANCEL_APPROVED);
        record2.setApplicationStatus(ApplicationStatus.CANCEL_PROCESSING);
        record3.setApplicationStatus(ApplicationStatus.DELETE);
        record4.setApplicationStatus(ApplicationStatus.DENIED);
        record5.setApplicationStatus(ApplicationStatus.PROCESSING);
        // 14. 对Mock的接口进行处理，定义接口的返回值
        doReturn(Lists.newArrayList(record1, record2, record3, record4)).when(signinOutRecordDao)
                .findByCompanyIdAndStaffIdAndSignTypeAndSignDateAndApplicationStatusIn(companyId, staffId,
                        signType, signDate, Lists.newArrayList(ApplicationStatus.APPROVED,
                                ApplicationStatus.WAITING_HR_APPROVAL, ApplicationStatus.PROCESSING));

        // 15. 执行私有函数进行测试
        Whitebox.invokeMethod(signService, "checkOutSign", signType, companyId, staffId, signDate);

        // 16. 校验Mock的对象的函数是否被调用了1次
        verify(signinOutRecordDao, times(1)).findByCompanyIdAndStaffIdAndSignTypeAndSignDateAndApplicationStatusIn(companyId, staffId,
                signType, signDate, Lists.newArrayList(ApplicationStatus.APPROVED,
                        ApplicationStatus.WAITING_HR_APPROVAL, ApplicationStatus.PROCESSING));
    }

    // 17. 该函数预计会产生Exception
    @Test(expected = IrenshiException.class)
    public void testCheckOutSign2() throws Exception {
        String staffId = "mockStaffId";
        SigninType signType = SigninType.SIGNIN_AFTERNOON;
        Date signDate = new Date();
        String companyId = "mockCompanyId";

        SigninOutRecord record = new SigninOutRecord();
        record.setApplicationStatus(ApplicationStatus.APPROVED);
        doReturn(Lists.newArrayList(record)).when(signinOutRecordDao)
                .findByCompanyIdAndStaffIdAndSignTypeAndSignDateAndApplicationStatusIn(companyId, staffId,
                        signType, signDate, Lists.newArrayList(ApplicationStatus.APPROVED,
                                ApplicationStatus.WAITING_HR_APPROVAL, ApplicationStatus.PROCESSING));

        Whitebox.invokeMethod(signService, "checkOutSign", signType, companyId, staffId, signDate);

        verify(signinOutRecordDao, times(1)).findByCompanyIdAndStaffIdAndSignTypeAndSignDateAndApplicationStatusIn(companyId, staffId,
                signType, signDate, Lists.newArrayList(ApplicationStatus.APPROVED,
                        ApplicationStatus.WAITING_HR_APPROVAL, ApplicationStatus.PROCESSING));
    }
}

```

## 代码详细分析

### 使用JUnit测试框架启动
```java
@RunWith(PowerMockRunner.class)
```
`@RunWith`是JUnit的注解，可以指定测试用的Runner。如：使用Spring上下文做测试的代码为`@RunWith(SpringJUnit4ClassRunner.class) `，使用纯Mockito的代码为`@RunWith(MockitoJUnitRunner.class)`

### 注入待测试的类
```java
@InjectMocks
private SignServiceImpl signService;
```
`@InjectMocks`是原生Mockito的注解，负责将待测试的类注入到单元测试中。这里需要注意：
1. 此处的对象（SignServiceImpl）必须是实体对象，不能是接口或者抽象类。因为`InjectMocks`需要实例化该对象
2. 对象中所有的依赖注入都会以一个简单粗暴的方式解决，默认将所有的`@Autowired`对象注入成`null`
所以，只要增加这个注解就可以快速生成一个对象，比Spring的Bean管理简单很多。

### Mock一个Bean
大部分情况下，我们还是要Mock一些Bean，来辅助完成单元测试的。
```java
@Mock
private SigninOutRecordDao signinOutRecordDao;
```
`@Mock`也是原生Mockito的注解，增加该Mock之后，`SignServiceImpl`所有依赖`SigninOutRecordDao`的地方，都会被注入成该对象。我们可以对Mock的对象进行各种操作，修改函数调用行为（称作Stub，有人叫“打桩”）等。

### 测试类包含私有函数的调用时
```java
@Test
@PrepareOnlyThisForTest(SignServiceImpl.class)
public void testSignV3() throws Exception 
```
`@Test`注解不用说，就是生成一个测试用例。`@PrepareOnlyThisForTest`需要特别注意。因为我们在测试`SignServiceImpl`的过程中，需要对`SignServiceImpl`的私有函数`checkOutSign`进行Stubbing，修改其行为，所以必须使用`@PrepareOnlyThisForTest(SignServiceImpl.class)`为Stubbing做好准备。

### 为测试实体Stubbing
测试的时候，我们需要用到实体类，但又不想使用实体类的所有实现函数。所以我们需要针对特定的某些函数进行Stubbing。
```java
SignServiceImpl spy = spy(signService);
```
对Mock的接口（如：SigninOutRecordDao signinOutRecordDao）来说，直接对其中的函数进行Stub即可。但如果要对测试实体进行Stubbing，则需要先对其进行`spy`。然后即可开展后边的Stubbing操作。

### 对函数进行Stubbing
先从对Mock对象进行的Stubbing开始。
```java
doReturn(Lists.newArrayList(record1, record2, record3, record4)).when(signinOutRecordDao)
                .findByCompanyIdAndStaffIdAndSignTypeAndSignDateAndApplicationStatusIn(companyId, staffId,
                        signType, signDate, Lists.newArrayList(ApplicationStatus.APPROVED,
                                ApplicationStatus.WAITING_HR_APPROVAL, ApplicationStatus.PROCESSING));
```
这个函数对`signinOutRecordDao`进行Stubbing。根据字面意思可以理解：

    This function will be stubbed as: **return** the given **List** when **signinOutRecordDao**
    is called by **findByCompanyIdAndStaffIdAndSignTypeAndSignDateAndApplicationStatusIn**
    with these **parameters**

都比较容易理解。

### 对私有函数进行Stubbing
对私有函数进行Stubbing和公共函数类似：
```java
doNothing().when(spy, "checkOutSign", signType, companyId, staffId, signDate);
```
在这里，Stubbing对象是实体`spy`的`checkOutSign`函数，参数为`signType, companyId, staffId, signDate`。

### 当被Stub的函数不是确定输入参数时
`actualSignV3`这个函数在调用的时候，用了一个很Anti-Pattern的一个设计，`signTime`这个参数用的是`new Date()`。暂且先不讨论代码的质量，先看看下边的Stub代码：
```java
doReturn(signinResult).when(spy).actualSignV3(eq(staffId), eq(signType), eq(wifiName), eq(wifiMac),
                eq(longitude), eq(latitude), eq(radius), eq(locationName), eq(mobileId), eq(signDate), eq(companyId),
                any(), eq(isSigninOnlyOnce), eq(false));
```
`any()`函数意思是，当`actualSignV3`函数调用的时候，无论`signTime`这个参数是什么值，这个Stubbing均生效。需要注意的是，一旦函数参数里边有任何一个`any`或类似的`Matcher`函数（如`anyInt`，`anyString`等）时，其他所有参数也必须以同样的形式出现。
上边代码中可以看到所有参数都使用了`eq()`进行封装。

### 另一种Stubbing方法（不推荐）
```java
when(spy.actualSignV3(eq(staffId), eq(signType), eq(wifiName), eq(wifiMac),
        eq(longitude), eq(latitude), eq(radius), eq(locationName), eq(mobileId), eq(signDate), eq(companyId),
        any(), eq(isSigninOnlyOnce), eq(false))).thenReturn(signinResult);
```
这种Stubbing比较符合汉语的语法：当xxx的时候，怎么怎么样。但是这样Stub有一个不好的地方，Stub的时候会首先执行`actualSignV3`的原版函数，然后再进行替换。可向而知，由于很多Bean都没有定义，直接抛`NullPointerException`。

### 执行测试代码
执行测试代码的方法和普通调用一样：
```java
SigninResult result = spy.signV3(staffId, signType, wifiName, wifiMac, longitude, latitude, radius, locationName, mobileId,
        signDate, companyId, isSigninOnlyOnce);
```
但这里仍有需要注意的地方：当调用的时候，只能使用被spy的对象`spy`，而不能使用原对象`signService`。因为只有`spy`被Stubbed了，而`signService`仍然保持不变。

### 校验函数调用情况
校验`checkOutSign`函数是否以给定的参数`signType, companyId, staffId, signDate`被调用了**一次**。
```java
verifyPrivate(spy, times(1)).invoke("checkOutSign", signType, companyId, staffId, signDate);
```

### 校验具有不确定参数的函数时
和Stubbing的时候一样，校验时如果有任意一个参数使用了`Matcher`形式，则其他所有函数都必须使用`Matcher`。
```java
verify(spy, times(1)).actualSignV3(eq(staffId), eq(signType), eq(wifiName), eq(wifiMac),
        eq(longitude), eq(latitude), eq(radius), eq(locationName), eq(mobileId), eq(signDate), eq(companyId),
        any(), eq(isSigninOnlyOnce), eq(false));
```

### 校验输出结果
这个没什么好说的
```java
assertTrue(signinResult == result);
```

### 对私有函数进行测试
私有函数测试的难点在于我们没有办法调用私有函数，但是PowerMock帮我们解决了这个问题。
```java
Whitebox.invokeMethod(signService, "checkOutSign", signType, companyId, staffId, signDate);
```
PowerMock使用`Writebox`，通过反射的方式调用`checkOutSign`这个函数。

### 正确运行会抛出异常
这个也没什么好说的，JUnit4原生的处理方式。
```java
@Test(expected = IrenshiException.class)
```

## 最后
一个框架+一个Mock+一个Mock增强，基本可以满足大部分单元测试的需求了，在配合使用Jenkins等CI工具，单元测试是要飞起来的节奏！
