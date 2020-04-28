---
date: 2020-03-20
title: "一个医师排班算法的实现"
description: "来自我的毕设中的难度最大的核心需求"
tags: ["java", "algorithm"]
categories: ["算法"]
---

**tips:点击左上角的爱心 ❤️ 可切换夜间模式保护眼睛哦！！**

本文旨在实现一个智能排班的算法

<!-- TOC -->

- [需求分析](#需求分析)
- [所需数据](#所需数据)
- [项目落地](#项目落地)
- [算法设计](#算法设计)
- [总结](#总结)

<!-- /TOC -->

> 毕设项目中核心需求是要根据医师的手术、请假、已预约时间还有医师的级别来进行自动排班，也是毕设中最具难度的一个需求

---

# 需求分析

- 用户要选择医师进行预约时，查询出该医师的排班时间，可用则显示**“可预约”**，否则不显示
- 医院查询**各个科室下的排班情况**，显示对应时间段是谁在坐诊
- 医师可以查询到**自己的排班情况**
- 查询的时限都是当天起未来一周 （即**今日周一，则最远可查询到下周一**）

---

# 所需数据

- 医师的 ID
- 医师的级别
- 手术或者请假的安排

---

# 项目落地

- 前台传来**医师 ID**或者**科室 ID**

  > 医院可以根据科室 ID 查看不同科室的排班表，而医生只能查询自己科室的排班表，最后用户根据医师 ID 和科室 ID 查询医师的个人排班表

  我的项目后台可以查询到发送查询请求的这个用户是医院、医师还是患者，所以判断查询科室排班还是医师排班不难

- 根据科室 ID 查询科室下的所有医师

  1. **提取关键医师信息，初始化一个数组表示医师时间表，把这些封装暂存**

  2. **按照医师级别进行分组，将级别相同的医师存入同一 List**

- 根据每个医生 ID 查询请假表，**获取手术或者请假的安排，根据请假表将对应医师的对应时间段置 0**

- 根据各个医师的排班表完成科室的排班，**优先选择高级别的医师填充科室排班表**

- 向前台返回对应的排班表

---

# 算法设计

- 建立一个对象封装必要的数据，如下

  ```java
  package com.shining.sys.commons.entity;

  import lombok.AllArgsConstructor;
  import lombok.Data;
  import lombok.NoArgsConstructor;

  /**
   * @program: shining2
   * @description: 医师排班表
   * @author: Dawn
   * @create: 2020-03-20
   */
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class ShiningSchedule {

      // 医师ID
      private Integer doctorId;

      // 医师姓名
      private String doctorName;

      // 医师级别
      private Integer level;

      // 科室ID
      private Integer deptId;

      // 排班时间表
      private Integer[] schedule;
  }

  ```

- 建立一个 Builder 类构造排班表

```java
// 注入Spring容器
@Component
public class ScheduleBuilder {

    @Autowired
    private IDoctorService doctor;

    private static IDoctorService doctorService;

    @Autowired
    private ILeaveService leave;

    private static ILeaveService leaveService;

    @PostConstruct
    public void init() {
        doctorService = doctor;
        leaveService = leave;
    }

}
```

> 这个 Builder 属于工具类，外部可调用的方法必须是静态的，可我们需要查询数据库来进行数据计算，Spring 容器没有办法为我们自动注入，所以选择命名两个私有变量，一个静态、一个非静态，在初始化这个类组件时将自动注入的非静态成员赋值给静态成员，这样就可以完成注入了，比如上面我的 doctorService、leaveService 两个需要注入的 service 组件就各建立了两个变量，使用@PostConstruct 注解调用初始化方法为静态的变量赋值

- 建立一个 map 存放信息`public static final Map<String, Object> SCHEDULE_MAP = new HashMap<>();`
- 建立两个方法供外部调用

```java
    /**
     * 拿到部门的排班表
     */
    public static List<ShiningSchedule> getDeptSchedule(Integer deptId) {
        buildDeptSchedule(deptId);
        return (List<ShiningSchedule>) SCHEDULE_MAP.get(DEPT_PREFIX + deptId);
    }

    /**
     * 拿到医师的排班表
     */
    public static ShiningSchedule getDoctorSchedule(Integer doctorId, Integer deptId) {
        buildDeptSchedule(deptId);
        return (ShiningSchedule) SCHEDULE_MAP.get(DOCTOR_PREFIX + doctorId);
    }

```

- 根据传来的医师、科室 ID 查询数据库拿到科室下所有医生的信息

  - 提取每个医师的 ID、姓名、级别、科室、加上下面这条描述的时间表数组一起封装到前面建立的`ShiningSchedule`对象中

    ```java
        /**
         * 初始化医师信息的map：key为doctor:info:医师id，value为数据库格式的医师信息
         */
        private static void getDoctorFromDept(Integer deptId) {
            List<Integer> doctorIds = doctorService.getIdByDeptId(deptId);
            for (Integer doctorId : doctorIds) {
                Doctor doctor = doctorService.getById(doctorId);
                // 只取出医师ID、医师姓名、级别信息、科室信息、并初始化排班数组，存入shiningSchedule实体类
                ShiningSchedule shiningSchedule = new ShiningSchedule(doctor.getId(), doctor.getName(), doctor.getLevel(), doctor.getDeptId(), initArray());
                // 将信息以doctor:医师ID为key，shiningSchedule为value存入map
                SCHEDULE_MAP.put(DOCTOR_PREFIX + doctor.getId(), shiningSchedule);
            }
            // 额外存入doctor的ID信息
            SCHEDULE_MAP.put(DOCTOR_ID_PREFIX, doctorIds);
        }

    ```

  - 每个医生都初始化一个大小为 16 的数组空间，对应 8 天每天上午下午的两个时间段，且全部初始化为**1-可用**，这些数组全部存放在医师对应的`ShiningSchedule`对象里，

    ```java
        /**
         * 返回一个全部为1-可用的空间长度为16的数组
         */
        private static Integer[] initArray() {
            return new Integer[]{1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1};
        }

    ```

  - 遍历医师的级别信息，将 4 个级别分成 4 个数组，各级别的医师 ID 存放在各级别的数组中，4 个数组存放在 key 为医师级别，value 为该级别数组的 map 中

    ```java
        /**
         * 初始化医师级别map：key为level:级别id，value为该级别的所有医师ID
         */
        private static void initLevelMap() {
            List<Integer> LEVEL_NO1 = new ArrayList<>();
            List<Integer> LEVEL_NO2 = new ArrayList<>();
            List<Integer> LEVEL_NO3 = new ArrayList<>();
            List<Integer> LEVEL_NO4 = new ArrayList<>();
            // 获取医师ID
            List<Integer> doctorIds = (List<Integer>) SCHEDULE_MAP.get(DOCTOR_ID_PREFIX);
            for (Integer doctorId : doctorIds) {
                //
                ShiningSchedule shiningSchedule = (ShiningSchedule) SCHEDULE_MAP.get(DOCTOR_PREFIX + doctorId);
                switch (shiningSchedule.getLevel()) {
                    case 1:
                        LEVEL_NO1.add(shiningSchedule.getDoctorId());
                        break;
                    case 2:
                        LEVEL_NO2.add(shiningSchedule.getDoctorId());
                        break;
                    case 3:
                        LEVEL_NO3.add(shiningSchedule.getDoctorId());
                        break;
                    case 4:
                        LEVEL_NO4.add(shiningSchedule.getDoctorId());
                        break;
                    default:
                        break;
                }
            }
            SCHEDULE_MAP.put(LEVEL_PREFIX + "1", LEVEL_NO1);
            SCHEDULE_MAP.put(LEVEL_PREFIX + "2", LEVEL_NO2);
            SCHEDULE_MAP.put(LEVEL_PREFIX + "3", LEVEL_NO3);
            SCHEDULE_MAP.put(LEVEL_PREFIX + "4", LEVEL_NO4);
        }

    ```

* 筛选请假表时间，另有安排的将该医生的该段时间更新为**0-不可用**

  > 遍历每个医生的时间安排，如果包含手术请假信息，判断该时间覆盖了数组空间的什么位置，这些位置全部置 0，关于变量 day 和对应数组位置的关系如下表

  |   下标\对应天数   | 第 1 天(今天) | 第 2 天 | 第 3 天 | 第 4 天 | 第 5 天 | 第 6 天 | 第 7 天 | 第 8 天 |
  | :---------------: | :-----------: | :-----: | :-----: | :-----: | :-----: | :-----: | :-----: | :-----: |
  | 排班表数组下标[]: |     0、1      |  2、3   |  4、5   |  6、7   |  8、9   | 10、11  | 12、13  | 14、15  |
  |   对应天数 day:   |       0       |    1    |    2    |    3    |    4    |    5    |    6    |    7    |

  说明：

  ```java
  int day = fromTime.getDate() - new Date().getDate();
  ```

  请假表的开始时间如果处于上午，那么获取结束时间相减得到持续时间，持续时间内涵盖的置 0，

  假设请假是 3 天，周一下午到周三上午 9 点，那么周一下午、周二一天、周三上午都应该算请假

  第一遍循环 duration_Day = 2，day[1]置 0、day[2]置 0，最后 duration_Day 自减 1

  第二遍循环 duration_Day = 1，判断是否是最后一天，是最后一天则根据结束时间来选择最后一天的上午下午是否置 0

  ```java
      /**
       * 根据请假表的信息填充各医师排班
       */
      private static void setLeaveTime() {
          // 获取当前的医师ID
          List<Integer> doctorIds = (List<Integer>) SCHEDULE_MAP.get(DOCTOR_ID_PREFIX);
          // 获取所有已经审核通过的查询时间内的请假或手术信息
          List<Leave> leaves = leaveService.getDoctorValidLeave(doctorIds);
          for (Leave leave : leaves) {
              Integer doctorId = leave.getDoctorId();
              ShiningSchedule doctorSchedule = (ShiningSchedule) SCHEDULE_MAP.get(DOCTOR_PREFIX + doctorId);
              // 获取排班表
              Integer[] schedule = doctorSchedule.getSchedule();
              // 拿到该医师的请假信息
              Date fromTime = leave.getFromTime();
              Date toTime = leave.getToTime();

              int fromHour = fromTime.getHours();

              // 判断开始时间距离今天是几天
              int day = fromTime.getDate() - new Date().getDate();

              // 拿到持续日期
              int durationDay = toTime.getDate() - fromTime.getDate();
              // 拿到持续小时
              int toHour = toTime.getHours();
              // 将排班表对应的时间设置为0-不可用
              if (fromHour <= 12) {// 请假从上午开始
                  while (durationDay > 0) {// 如果持续时间大于一天
                      schedule[day * 2] = 0;// 将上午置0
                      schedule[day++ * 2 + 1] = 0;// 下午置0
                      --durationDay;// 持续天数自减
                  }
                  // 开始处理最后一天
                  if (durationDay == 0) { // 持续时间仅当天，且包含下午
                      if (toHour > 8 && toHour < 12)
                          schedule[day * 2] = 0; // 持续时间仅上午
                      if (toHour >= 12) {
                          // 持续时间包含下午
                          schedule[day * 2] = 0;
                          schedule[day * 2 + 1] = 0;
                      }
                  }
                  // 结束处理最后一天
              } else {// 请假从下午开始
                  schedule[day++ * 2 + 1] = 0;// 先将下午置0
                  while (durationDay > 0) {// 如果持续时间大于一天
                      // 开始处理最后一天
                      if (durationDay == 1 && toHour > 8 && toHour <= 12) {
                          schedule[day * 2] = 0;// 将上午置0
                          break;
                      }
                      // 结束处理最后一天
                      schedule[day * 2] = 0;// 将上午置0
                      schedule[day++ * 2 + 1] = 0;// 下午置0
                      --durationDay;// 持续天数自减
                  }
              }
              // 更新map信息
              doctorSchedule.setSchedule(schedule);
              SCHEDULE_MAP.put(DOCTOR_PREFIX + doctorId, doctorSchedule);
          }
      }

  ```

* 对科室进行排班

  ```java
      /**
       * 根据科室中的医师排班表排出科室时间表
       */
      private static void setFinalSchedule(Integer deptId) {
          // 获取所有信息
          // 这是一个部门时间表
          List<ShiningSchedule> deptFinalSchedule = new ArrayList<>();
          // 初始化一个数组用来暂存被安排的医师
          Integer[] deptScheduleDoctorId = new Integer[]{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0};

          // 取出等级信息
          List<Integer> LEVEL_NO1 = (List<Integer>) SCHEDULE_MAP.get(LEVEL_PREFIX + "1");
          List<Integer> LEVEL_NO2 = (List<Integer>) SCHEDULE_MAP.get(LEVEL_PREFIX + "2");
          List<Integer> LEVEL_NO3 = (List<Integer>) SCHEDULE_MAP.get(LEVEL_PREFIX + "3");
          List<Integer> LEVEL_NO4 = (List<Integer>) SCHEDULE_MAP.get(LEVEL_PREFIX + "4");

          for (int i = 0; i < deptScheduleDoctorId.length; i++) {

              // 根据从高等级到低等级的顺序找到一位时间可用的医生
              if (LEVEL_NO1 != null && LEVEL_NO1.size() > 0) {// 判断该等级下是否有医生
                  // 判断该等级医生是否时间有效
                  buildSchedule(LEVEL_NO1, deptScheduleDoctorId, i);
              }

              // 根据从高等级到低等级的顺序找到一位时间可用的医生
              if (LEVEL_NO2 != null && LEVEL_NO2.size() > 0) {// 判断该等级下是否有医生
                  // 判断该等级医生是否时间有效
                  buildSchedule(LEVEL_NO2, deptScheduleDoctorId, i);
              }

              // 根据从高等级到低等级的顺序找到一位时间可用的医生
              if (LEVEL_NO3 != null && LEVEL_NO3.size() > 0) {// 判断该等级下是否有医生
                  // 判断该等级医生是否时间有效
                  buildSchedule(LEVEL_NO3, deptScheduleDoctorId, i);
              }

              // 根据从高等级到低等级的顺序找到一位时间可用的医生
              if (LEVEL_NO4 != null && LEVEL_NO4.size() > 0) {// 判断该等级下是否有医生
                  // 判断该等级医生是否时间有效
                  buildSchedule(LEVEL_NO4, deptScheduleDoctorId, i);
              }
          }
          // 已经完成科室内的排班，接下来向list中封装
          for (Integer doctorId : deptScheduleDoctorId) {
              ShiningSchedule doctor = (ShiningSchedule) SCHEDULE_MAP.get(DOCTOR_PREFIX + doctorId);
              deptFinalSchedule.add(doctor);
          }
          SCHEDULE_MAP.put(DEPT_PREFIX + deptId, deptFinalSchedule);
      }

      /**
       * 对某一级别中的医师进行排班
       */
      private static void buildSchedule(List<Integer> LEVEL, Integer[] deptScheduleDoctorId, Integer i){
          for (Integer doctorId : LEVEL) {
              // 取出医生
              ShiningSchedule doctor = (ShiningSchedule) SCHEDULE_MAP.get(DOCTOR_PREFIX + doctorId);
              // 取出时间表
              Integer[] schedule = doctor.getSchedule();
              // 如果这段时间已安排医师，那么将此医师的这段时间置0，否则不变
              schedule[i] = deptScheduleDoctorId[i] != 0 ? 0 : schedule[i];

              // 如果此医师这段时间有空，那么将医师ID暂存，否则跳过这个医师
              deptScheduleDoctorId[i] = schedule[i] == 1 ? doctorId : deptScheduleDoctorId[i];

              // 更新map
              doctor.setSchedule(schedule);
              SCHEDULE_MAP.put(DOCTOR_PREFIX + doctorId, doctor);
          }
      }

  ```

> 附上请假表 service 层的查询有效请假单的代码

```java
    @Override
    public List<Leave> getDoctorValidLeave(List<Integer> doctorIds) {
        // 获取当前时间
        Date today = Timestamp.valueOf(
                new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.SIMPLIFIED_CHINESE).format(new Date())
        );
        // 从当天0点查起
        today.setHours(0);
        today.setMinutes(0);
        today.setSeconds(0);
        QueryWrapper<Leave> wrapper = new QueryWrapper<>();
        // 查询已通过的
        wrapper.eq("status", DataSourcesConstants.LEAVE_PASS);
        // 请假开始时间范围 从今天的零点开始
        wrapper.ge("fromTime", today);
        // 查到下周
        Date dateTo = Timestamp.valueOf(
                new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.SIMPLIFIED_CHINESE).format(new Date())
        );
        dateTo.setDate(today.getDate() + 8);
        dateTo.setHours(0);
        dateTo.setMinutes(0);
        dateTo.setSeconds(0);
        wrapper.lt("fromTime", dateTo);
        // 查询指定医生的请假单
        wrapper.in("doctorId", doctorIds);
        return baseMapper.selectList(wrapper);
    }

```

对于 Java 中的 Date()类型的 API 我也不太熟悉，中间出过一个时间戳不正确的 bug，查不到对应时间的请假单，才知道问题出在 `new Date()`的代码上，Java 直接 new 一个 Date 对象是会有毫秒值的，而我希望可以从 0 点开始查询当天的有效请假单，百度了一下才知道`Timestamp.valueOf(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.SIMPLIFIED_CHINESE).format(new Date())`是可以格式化 Date()对象的，使用`"yyyy-MM-dd HH:mm:ss"`来舍弃毫秒字段，就能生成一个 ~~没有毫秒的~~ 毫秒数为 0 的 Date()对象了

---

# 总结

> 我的这段代码其实还有很多地方可以优化
>
> - 可以试着不建立级别数组，而是根据 ID、级别两个数字来构建医师信息对象的 map key
> - 医师的级别是一个绝对优先级，也就是说假如高级别的没有请假或者手术安排，那么排班是不会轮到低级别的医师的
>
> 等等……
>
> 总之代码还是要多敲多练才能更熟练地解决实际生产中的需求问题，加油！
