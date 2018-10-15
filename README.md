# VIPKID international open platform

#### 前言 소개

#### 修订时间 수정 시간

|时间  시간        |版本 버전    | 修订者   수정자   |功能 기능             |备注   비고      |
|:-----------|:------|:----------|:-----------------|:------------|
|2018-09-06  |V1.0   | |获取来电用户信息 발신자 정보 가져 오기    |-            |


#### 账号申请 계정 신청

账号申请联系人계정 신청 담당자：[international_rd@vipkid.com.cn](mailto:international_rd@vipkid.com.cn) 

1. 开发前请先申请beta测试账号개발 전에 beta 테스트 계정을 신청하십시오.
2. 联调通过后，再提供相关资料申请生产环境的账号
연동조절 통과 후, 관련 정보를 제공한 다음  생산 환경과 관련된 계정을 신청하십시오.
3. 账号开通后包含以下信息：계정을 신청하시면 다음과 같은 정보가 있습니다.
```text
账号code 계정 코드：accountCode
签名密钥싸인 키：signKey
```    
    
#### 接口标准 인터페이스 표준

##### 基本要求 기본 요구/ Requirements

1. 调用接口前请先申请账号 인터페이스 접속 전 계정을 신청하세요. 
2. 接口统一使用UTF-8编码格式 인터페이스는 통일로 UTF-8 인코딩 형식을 사용합니다.
3. 所有接口必须进行签名验证(签名不区分大小写) 확인을 위해 모든 인터페이스에 서명해야합니다 (서명은 대소 문자를 구분하지 않음)

##### 消息体 / Response body

```json
{
    "code": 200,
    "msg": "",
    "data": null
}
```

|名称    명칭    |类型      유형   |示例   예     |备注             |비고|
|:----------|:------------|:----------|:---------------|
|code       |Integer      |200        |返回200时代表成功，其它的代表失败|200으로 돌아가면 성공, 기타는 실패|
|msg        |String       |           |错误提示信息| 오류 정보 알림|
|data       |Object/Array |           |返回的具体数据|돌아온 구체적인 데이터|

##### 接口域名 / Domain

beta环境beta환경（Beta）: https://pre-openapi.vipkid.com

生产环境생산 환경(Production)：https://openapi.vipkid.com

##### 请求头 / Http Headers

以下的http header信息, 每次调用接口时都需要传递

다음 http header 정보는 인터페이스가 호출 될 때마다 전달되어야 합니다.

|名称 명칭             |含义   함의        |示例  예             |
|:-------------------|:-------------- |:------------------|
|vk-cr-code          |国家区号      국가 코드   |kr                 |                          
|vk-account-code     |开通账号后分配的계정 개설 후 분배된accountCode|89887780788                   |                 
|vk-req-timestamp    |请求时间戳,也是计算签名时使用的 서명을 계산할 때도 사용됨 timestamp |1536206825036       |                 
|vk-sign             |签名值 싸인치          |84d9cfc2f395ce883a41d7ffc1bbcf4e|

##### 签名验证 / Sign 싸인

对于非json方式传参的接口按照以下方式进行签名计算：
비 json 모드 변수에 대한 인터페이스는 다음과 같은 방식으로 계산됩니다.

1. 将所有的参数，按照参数名正序排序，然后按照 모든 매개변수를 변수 이름 순서로 정렬 한 후 param1=value1& param2=value2的形式排序의 형식으로 순서를 정한다.，如예:：a=1, c=2, b=3 则拼接后的字符串为그다음 스티치 된 문자열은 a=1&b=3&c=2
2. 将时间戳拼接到第一步拼接的字符串后面，如:
첫 번째 단계에서 스티치 된 문자열 다음에 타임 스탬프를 연결합니다. 예: a=1&b=3&c=2&timestamp=1536206825036
3. 将签名密钥拼接到第二步拼接的字符串后面，如：
두 번째 단계에서 스티칭 된 문자열에 서명 키 연결합니다. 
예:a=1&b=3&c=2&timestamp=1536206825036&signKey=AfqPeRds
4. 将第三步拼接的字符串进行md5的到32位的签名值，然后放入请求header中，对应vk-sign 
세 번째 단계의 문자열은 md5의 32 비트 서명 값에 연결되고 vk-sign에 해당하는 요청 헤더에 배치됩니다
5. 签名计算时，不包含空值参数 서명계산시 null 매개 변수가 포함되지 않습니다.
6. 签名值不区分大小写 서명값은 대소 문자를 구분하지 않습니다.

```java
/**
 * generate sign
 * @param params request params
 * @param signKey account`s signKey
 * @return sign
 */
public static String genetateSign(Map<String, String> params, String signKey, Long timestamp) {
    TreeMap<String, String> treeMap = new TreeMap<>();
    treeMap.putAll(params);
    StringBuilder builder = new StringBuilder();
    for (Map.Entry<String, String> entry : treeMap.entrySet()) {
        if (null != entry.getValue() && !entry.getValue().trim().equals("")) {
            builder.append(entry.getKey() + "=" + entry.getValue() + "&");
        }
    }
    builder.append("timestamp=" + timestamp + "&");
    builder.append("signKey=" + signKey);
    System.out.println(builder.toString());
    return MD5Util.toMD5(builder.toString());
}
```

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.UnsupportedEncodingException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;


public class MD5Util {

    private static final Logger logger = LoggerFactory.getLogger(MD5Util.class);
    
    private static final String UTF8 = "utf-8";

    public static String md5Digest(String src) {
        try {
            // 定义数字签名方法디지털 서명방법 정의, 可用가용：MD5, SHA-1
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] b = md.digest(src.getBytes(UTF8));
            return byte2HexStr(b);
        } catch (Exception e) {
            logger.error("md5Digest error");
        }
        return null;
    }

    private static String byte2HexStr(byte[] b) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < b.length; i++) {
            String s = Integer.toHexString(b[i] & 0xFF);
            if (s.length() == 1) {
                sb.append("0");
            }
            sb.append(s.toUpperCase());
        }
        return sb.toString();
    }

    private static final char[] DIGITS = { '0', '1', '2', '3', '4', '5', '6', '7',
     '8', '9', 'A', 'B', 'C', 'D', 'E', 'F' };

    public static String toMD5(String text) {
        return toMD5(text, "utf-8");
    }

    public static char[] encodeHex(byte[] data) {
        int l = data.length;
        char[] out = new char[l << 1];
        // two characters form the hex value.
        int j = 0;
        for (int i = 0; i < l; i++) {
            out[j++] = DIGITS[(0xF0 & data[i]) >>> 4];
            out[j++] = DIGITS[0x0F & data[i]];
        }
        return out;
    }

    public static String toMD5(String text, String charset) {
        MessageDigest msgDigest = null;
        try {
            msgDigest = MessageDigest.getInstance("MD5");
        } catch (NoSuchAlgorithmException e) {
            throw new IllegalStateException("System doesn't support MD5 algorithm.");
        }
        try {
            msgDigest.update(text.getBytes(charset));
        } catch (UnsupportedEncodingException e) {
            throw new IllegalStateException("System doesn't support your EncodingException.");
        }
        byte[] bytes = msgDigest.digest();
        String md5Str = new String(encodeHex(bytes));
        return md5Str;
    }
}
```

#### 接口列表인터페이스 목록 / Api List

##### 获取来电用户信息발신자 정보 가져 오기 / getParentByPhone

###### 接口说明인터페이스 설명

根据来电号码获取家长信息、学生信息、课程信息、订单信息

발신자 번호를 기준으로 학부모 정보, 학생 정보, 수업 정보, 주문 정보를 확인.

###### 接口地址 인터페이스 주소 / Api Path

|名称      명칭     |类型         유형                        |
|:-------------|-------------------------------------|
|path          |/api/parent/getParentByPhone            |
|method        |GET                          |

?> eg: https://openapi.vipkid.com/api/parent/getParentByPhone

###### 接口入参 / Request

|Name | Type | Example | Comments |
|:------------- | --- | --- | --- |
| callPhone | String | 来电号码발신자 번호 | 必传，以82开头82를 시작으로 |
| classBeginTime | Long | 1536206825036 | 时间戳，上课开始时间，默认2周前 수업 시작 시간 2주전을 기본값으로 설정 |
| classEndTime | Long | 1536206825036 | 时间戳，上课结束时间，默认当前时间 수업종료 시간을 현재시간으로 설정 |
| orderBeginTime | Long | 1536206825036 | 时间戳，订单创建开始时间，默认2周前 주문생성 시작 시간 기본값을 2주전으로 설정|
| orderEndTime | Long | 1536206825036 | 时间戳，订单创建结束时间，默认当前时间 주문 종료 시간을 현재 시간으로 설정|

###### 返回值 / Response

家长信息 학부모 정보

| Name | Type | Example | Comments |
| --- | --- | --- | --- |
| data.parentId | Long | 12320947 | 家长Id 학부모ID|
| data.countryCode | String | 82 | 电话区号지역번호 |
| data.registerPhone | String | 1029872233 | 电话号 전화번호|
| data.callPhone | String | 1029872233 | 来电号码 발신자번호|
| data.status | String | TEST | 用户状态,参见字典值사용자 상태  UserStatus |
| data.sessionInventories | Array |   | 家长的课时信息학부모 수업 정보 |
| data.students | Array |   | 学生信息학생 정보
| data.orders | Array |   | 家长的订单信息학부모 구매 정보|

家长课时信息학부모 수업 정보，대응对应data.sessionInventories

| 名称 명칭| 类型 유형 | 示例 예 | 备注 비고|
| --- | --- | --- | --- |
| courseType | String | MAJOR | 课程类型 수업 유형|
| inventoryQuantity | Integer | 48 | 总课时 총 수업 |
| occupyQuantity | Integer | 5 | 预占课时예정 수업 |
| consumedQuantity | Integer | 34 | 已消费课时 이미 들은 수업 |
| availableQuantity | Integer | 9 | 剩余课时 남은 수업|
| unavailableQuantity | Integer | 0 | 不可用课时이용 불가능 수업(退费时锁定환불시) |

学生信息학생 정보，对应data.students

| 名称 명칭| 类型 유형 | 示例 예 | 备注 비고|
| --- | --- | --- | --- |
| studentId | Long | 12328605 | 学生Id 학생 Id|
| studentName | String | 구아인 | 学生姓名 학생 성명 |
| studentEnglishName | String | GUAIN | 英文姓名 영문 성명 |
| birthday | Long | 1470412800000 | 时间戳，出生日期 생년월일 |
| gender | Integer | 0 | 性别,参见字典值 Gender 성별 |
| learningProgresss | Array |   | 学生课时进度 학생 수업 진행 속도|

学生进度信息학생 진도 정보，对应learningProgresss

| 名称 명칭 | 类型 유형| 示例 예 | 备注 비고 |
| --- | --- | --- | --- |
| courseId | Long | 597816 | 课程类型Id 수업유형 Id |
| courseType | String | MAJOR | 课程类型 수업 유형  |
| lessonSn | String | MC-L3-U2-LC1-6 | 课标 커리큘럼 |
| level | String | L3-U2 | 级别 레벨 |
| classes | Array |   | 上课信息 수업정보 |

学生上课信息학생 수업 정보，对应classes

| 名称 명칭| 类型 유형 | 示例 예 | 备注 빅비고 |
| --- | --- | --- | --- |
| classId | Long | 1009 | 上课记录Id 수업기록 Id |
| teacherId | Long | 8090192 | 老师Id 선생님 Id |
| teacherName | String | Andy | 老师姓名 선생님 성명 |
| classTime | Long | 1531278000000 | 时间戳，上课时间 수업시간 |
| classStatus | String | COMPLETED | 上课状态수업 상태,参见字典值ClassStatus |
| classSubStatus | String | COMPLETED | 上课子状态,参见字典值classSubStatus |
| studentClassStatus | String | COMPLETED | 学生侧状态,参见字典值classSubStatus |
| teacherClassStatus | String | COMPLETED | 家长侧状态,参见字典值classSubStatus |
| bookTime | Long | 1529982169000 | 时间戳，预约时间 |
| studentEnterTime | Long | 1531277700000 | 时间戳，学生进入时间 |
| teacherEnterTime | Long | 1531274880000 | 时间戳，老师进入时间 |

家长订单信息 학부모 구매 정보，对应data. orders

| 名称 명칭 | 类型 유형 | 示例 예| 备注 비고 |
| --- | --- | --- | --- |
| orderNo | String |   | 订单号 주문 번호|
| status | Integer |   | 订单状态주문 상태 ,参见字典值OrderStatus |
| payTime | Long |   | 时间戳，支付时间 지불시간|
| refundTime | Long |   | 时间戳，退款时间 환불시간|
| source | Integer |   | 订单来源 구매 정보 |
| createTime | Long |   | 时间戳，订单创建时间주문 생성 시간  |
| orderDetails | Array |   | 订单明细 구매내역 |

订单明细 구매내역，对应orderDetails

| 名称 명칭 | 类型 유형 | 示例 예 | 备注 비고 |
| --- | --- | --- | --- |
| productTypeId | Long | 1 | 课程类型Id 수업유형Id |
| productTypeName | String | 商品包상품패키지 | 课程类型 수업유형 |
| productTypeShowName | String | 상품패키지 | 课程类型显示名称 수업유형 명칭 표시|
| productId | Long | 2 | 商品Id 상품Id |
| productName | String | 1单元12节课 1단원 12강| 商品名称 상품명칭 |
| productShowName | String | 1단원 12강 | 商品显示名称 상품명 표시 |
| productDescription | String |   | 商品描述 상품서술 |
| productCount | Integer | 1 | 数量 수량 |
| productDetails | Array |   | 商品明细 상품명세 |

商品明细 상품명세，对应productDetails

| 名称 명칭| 类型 유형| 示例 예| 备注 비고|
| --- | --- | --- | --- |
| productTypeId | Long | 2 | 课程类型Id 수업유형 Id |
| productTypeName | String | 主修课课时 정규수업시간| 课程类型 수업 유형|
| productTypeShowName | String | 정규수업시간 | 课程类型显示名称 수업유형 명칭 표시|
| productId | Long | 2 | 商品Id 상품 Id |
| productName | String | 1单元12节课 1단원 12강 | 商品名称 상품 명칭 |
| productShowName | String | 1단원 12강 | 商品显示名称 상품명 표시 |
| productDescription | String |   | 商品描述 상품 묘사 |
| productCount | Integer | 1 | 商品数量 상품 수량 |
| productUnit | String | 25分钟/节 25분/1교시| 商品单位 상품 단위|

###### Example

 headers
```json
{
    "vk-cr-code": "kr",
    "vk-account-code": "89887780788",
    "vk-req-timestamp": "1536206825036",
    "vk-sign": "84d9cfc2f395ce883a41d7ffc1bbcf4e"
}
```

parameter

```text
https://openapi.vipkid.com/api/parent/getParentById?callPhone=
18610657654&classBeginTime=&classEndTime=
&orderBeginTime=&orderEndTime=
```

response
```json
{
    "msg": "ok",
    "code": 200,
    "data": {
        "parentId": 12320947,
        "countryCode": "82",
        "registerPhone": "1029872233",
        "callPhone": "1029872233",
        "status": "TEST",
        "sessionInventories": [
            {
                "courseType": "MAJOR",
                "inventoryQuantity": 48,
                "occupyQuantity": 5,
                "consumedQuantity": 34,
                "availableQuantity": 9,
                "unavailableQuantity": 0
            },
            {
                "courseType": "TRIAL",
                "inventoryQuantity": 3,
                "occupyQuantity": 2,
                "consumedQuantity": 0,
                "availableQuantity": 1,
                "unavailableQuantity": 0
            }
        ],
        "students": [
            {
                "studentId": 12328605,
                "studentName": "구아인",
                "studentEnglishName": "GUAIN",
                "birthday": 1470412800000,
                "gender": 0,
                "learningProgresss": [
                    {
                        "courseId": 597816,
                        "courseType": "MAJOR",
                        "lessonSn": "MC-L3-U2-LC1-6",
                        "level": 3,
                        "classes": [
                            {
                                "classId": 1009,
                                "teacherId": 8090192,
                                "teacherName": "teacherName",
                                "classTime": 1531278000000,
                                "classStatus": "COMPLETED",
                                "classSubStatus": "COMPLETED",
                                "studentClassStatus": "COMPLETED",
                                "teacherClassStatus": "COMPLETED",
                                "bookTime": 1529982169000,
                                "studentEnterTime": 1531277700000,
                                "teacherEnterTime": 1531274880000
                            }
                        ]
                    }
                ]
            }
        ],
        "orders": [
            {
                "orderNo": "KR2320124180502d300e",
                "status": 2,
                "payTime": 1525233664000,
                "refundTime": 1525234264000,
                "source": 1,
                "createTime": 1525233004000,
                "orderDetails": [
                    {
                        "productTypeId": 1,
                        "productTypeName": "商品包",
                        "productTypeShowName": "상품 패키지",
                        "productId": 2,
                        "productName": "1单元12节课",
                        "productShowName": "1단원 12강",
                        "productDescription": "",
                        "productCount": 1,
                        "productDetails": [
                            {
                                "productTypeId": 2,
                                "productTypeName": "主修课课时정규수업",
                                "productTypeShowName": "정규수업 시간",
                                "productId": 2,
                                "productName": "1单元12节课 ",
                                "productShowName": "1단원 12강",
                                "productDescription": "",
                                "productCount": 1,
                                "productUnit": "25分钟/节"
                            }
                        ]
                    }
                ]
            }
        ]
    }
}
```

#### 字典值 / Enums

用户状态 회원 상태 / UserStatus

| 值 | 含义 |
| --- | --- |
|NORMAL | 正常用户 일반 회원|
|TEST | 测试用户 테스트 회원|

性别 성별 / Gender

| 值 | 含义 |
| --- | --- |
| 0| boy|
| 1| girl|

上课状态 / ClassStatus

| 值 | 含义 |
| --- | --- |
|BOOKING|预约中 예약 중|
|BOOKED|已预约 예약 완료|
|COMPLETED|已完课 수업 끝|
|INCOMPLETE|未完课 수업 중|
|CANCELED|已取消 취소|

上课状态 / ClassSubStatus

| 值 | 含义 |
| --- | --- |
|STUDENT_BOOKED|学生已预订 학생이 예약하였습니다. |
|STUDENT_CANCEL|学生取消 학생이 취소하였습니다.|
|TEACHER_CANCEL|老师取消 선생님께서 취소하셨습니다.|
|STUDENT_CANCEL_24H|学生24小时外取消 학생이 24시간 전 취소하였습니다.|
|STUDENT_CANCEL_8H_24H|学生8~24小时内取消 학생이 8~24시간 내 취소하였습니다.|
|STUDENT_CANCEL_8H|学生8小时内取消 학생이 8시간 내 취소하였습니다. |
|TEACHER_CANCEL_24H|老师24小时外取消 선생님께서 24시간 전 취소하셨습니다.|
|TEACHER_CANCEL_8H_24H|老师8~24小时内取消 선생님께서 8~24식시간 내 취소하셨습니다.|
|TEACHER_CANCEL_8H|老师8小时内取消 선생님께서 8시간 내 취소하셨습니다.|
|STUDENT_NO_SHOW|学生未进入教室 학생이 교실로 들어오지 않았습니다. |
|STUDENT_IT_PROBLEM|学生IT问题 학생 IT문제 |
|TEACHER_NO_SHOW|教师未进入教室 선생님께서 교실로 들어오시지 않았습니다.  |
|TEACHER_IT_PROBLEM|教室IT问题 교실 IT문제|
|SYSTEM_PROBLEM|系统问题 시스템 문제|
|BOOKING|预约中예약 중|
|COMPLETED|已完课 수업 완료|
|INCOMPLETE|未完课 수업 미완료|

订单状态 / OrderStatus

| 值 | 含义 |
| --- | --- |
|0|已关闭 닫힘|
|1|新创建 새로 만들기 |
|2|待支付 결제 대기|
|3|支付中 결제 중|
|4|支付完成 결제 완료|
|5|退费发起 환불 진행|
|6|退费拒绝 환불 거절|
|7|退费中 환불 중|
|8|退费取消 환불 취소|
|9|退费完成 환불 완료|
|10|已过期 만료됨|
