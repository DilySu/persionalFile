# BigDecimal 的正确使用方式

## 一、背景

BigDecimal 平时主要用于计算金钱时，其自身提供了很多的构造方法，但是这些构造方法使用不当会造成精度丢失，从而引起事故。

## 二、事故案例

### 1、问题

收银台计算商品价格报错，导致订单无法支付

### 2、问题复现

```java
public static void main(String[] args) {
    BigDecimal bigDecimal=new BigDecimal(88);
    System.out.println(bigDecimal);
    bigDecimal=new BigDecimal("8.8");
    System.out.println(bigDecimal);
    bigDecimal=new BigDecimal(8.8);
    System.out.println(bigDecimal);
}
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnTbPYc83lTspNoMUyOlJYgoCphKalAXSAYg4JmpeGJzdNQSYHdLODMbqjIXLbiasib05BSHSJOgNNgw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### 3、源码分析

```java
public static long doubleToLongBits(double value) {
    long result = doubleToRawLongBits(value);
    // Check for NaN based on values of bit fields, maximum
    // exponent and nonzero significand.
    if ( ((result & DoubleConsts.EXP_BIT_MASK) ==
          DoubleConsts.EXP_BIT_MASK) &&
         (result & DoubleConsts.SIGNIF_BIT_MASK) != 0L)
        result = 0x7ff8000000000000L;
    return result;
}
```

问题就处在 doubleToRawLongBits 这个方法上，在 jdk 中 double 类（float 与 int 对应）中提供了 double 与 long 转换，doubleToRawLongBits 就是将 double 转换为 long，这个方法是原始方法（底层不是 java 实现，是 c++ 实现的）。

### 4、原因分析

在 java 中 BigDecimal 处理数据时把十进制小数扩大 N 倍让它在整数上进行计算，并保留相应的精度信息。

1. float 和 double 类型，主要是为了科学计算和工程计算而设计的，之所以执行二进制浮点运算，是为了在广泛的数值范围上提供较为精确的快速近和计算。
2. 并没有提供完全精确的结果，所以不应该被用于精确的结果的场合。
3. 当浮点数达到一定大的数，就会自动使用科学计数法，这样的表示只是近似真实数而不等于真实数。
4. 当十进制小数位转换二进制的时候也会出现无限循环或者超过浮点数尾数的长度。

## 三、总结

在设计到精度计算时，我们尽量使用 String 类型来进行转换，而且涉及到 BigDecimal 的计算，要使用其对应方法进行计算。

## 四、工具类

这里封装一个 BigDecimal 工具类

```java
public class BigDecimalUtils {
    /**
     * double 加
     *
     * @param v1 加数
     * @param v2 加数
     * @return 和
     */
    public static BigDecimal doubleAdd(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.add(b2);
    }

    /**
     * float 加
     *
     * @param v1 加数
     * @param v2 加数
     * @return 和
     */
    public static BigDecimal floatAdd(float v1, float v2) {
        BigDecimal b1 = new BigDecimal(Float.toString(v1));
        BigDecimal b2 = new BigDecimal(Float.toString(v2));
        return b1.add(b2);
    }

    /**
     * double 减
     *
     * @param v1 被减数
     * @param v2 减数
     * @return 差
     */
    public static BigDecimal doubleSub(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.subtract(b2);
    }

    /**
     * float 减
     *
     * @param v1 被减数
     * @param v2 减数
     * @return 差
     */
    public static BigDecimal floatSub(float v1, float v2) {
        BigDecimal b1 = new BigDecimal(Float.toString(v1));
        BigDecimal b2 = new BigDecimal(Float.toString(v2));
        return b1.subtract(b2);
    }

    /**
     * double 乘
     *
     * @param v1 因数
     * @param v2 因数
     * @return 积
     */
    public static BigDecimal doubleMul(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.multiply(b2);
    }

    /**
     * float 乘
     *
     * @param v1 因数
     * @param v2 因数
     * @return 积
     */
    public static BigDecimal floatMul(float v1, float v2) {
        BigDecimal b1 = new BigDecimal(Float.toString(v1));
        BigDecimal b2 = new BigDecimal(Float.toString(v2));
        return b1.multiply(b2);
    }

    /**
     * double 除
     *
     * @param v1 被除数
     * @param v2 除数
     * @return 商
     */
    public static BigDecimal doubleDiv(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        // 保留小数点后两位 ROUND_HALF_UP = 四舍五入
        return b1.divide(b2, 2, RoundingMode.HALF_UP);
    }

    /**
     * float 除
     *
     * @param v1 被除数
     * @param v2 除数
     * @return 商
     */
    public static BigDecimal floatDiv(float v1, float v2) {
        BigDecimal b1 = new BigDecimal(Float.toString(v1));
        BigDecimal b2 = new BigDecimal(Float.toString(v2));
        // 保留小数点后两位 ROUND_HALF_UP = 四舍五入
        return b1.divide(b2, 2, RoundingMode.HALF_UP);
    }

    /**
     * double<br>
     * 比较v1 v2大小
     *
     * @param v1
     * @param v2
     * @return v1>v2 return 1  v1=v2 return 0 v1<v2 return -1
     */
    public static int doubleCompareTo(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.compareTo(b2);
    }

    /**
     * float<br>
     * 比较v1 v2大小
     *
     * @param v1
     * @param v2
     * @return v1>v2 return 1  v1=v2 return 0 v1<v2 return -1
     */
    public static int floatCompareTo(float v1, float v2) {
        BigDecimal b1 = new BigDecimal(Float.toString(v1));
        BigDecimal b2 = new BigDecimal(Float.toString(v2));
        return b1.compareTo(b2);
    }
}
```