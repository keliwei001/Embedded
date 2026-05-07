###### 	压力传感器

​	压力传感器读值，向板端输出0~5V电压，板端分压后，adc读取到的电压为传感器输出的一半即0~2.5V，芯片通过spi1和adc进行通信，读取到adc值后转换为压力输出到显示端

adc值：0~4095

电源：0~3.88888

压力范围：-20~20kpa

adc识别到传感器电压范围：0~2.5V	0V——（-20kpa），1.25V——0kpa，2.5V——20kpa



###### 	液位传感器

​	液位传感器读值，向板端输出0~3V电压，板端分压后，adc读取到的电压为传感器输出的一半即0~1.5V，芯片通过spi1和adc进行通信，读取到adc值后转换为压力输出到显示端，线性的

adc值：0~4095

电源：0~3.88888

液位范围：0~100%

adc识别到传感器电压范围：0~1.5	0V——0%，0.75V——50%，1.5V——100%



###### 压力过大连接ct5544打开制冷

问题：压力变化ct5544制冷无反应

原因：参数传输错误，内核中使用get_value_ain1，但是具体实现时用的value，并且在pressure_operation_status_with_value中使用value或get_value_ain1值去使用convert_to_kilo_ceil函数转换为压力值后，带有小数点，在返回值时出错

解决：先将参数改为get_value_ain1，在函数中直接转化为具体压力值，不使用convert_to_kilo_ceil函数

​	**使用printf打印相关日志**

​	读取sd卡中配置，读取传感器的电压——9866——》屏幕显示，ct5544控制

```

sd_window_main.c中检查压力：pressure_operation_statu()判断开关>>>>>>pressure_voltage.c中具体实现pressure_operation_status_with_value()>>>>>>

void pressure_operation_status_with_value(uint32_t value) {
    printf("-------------------------------------------------------------------------------get_value_ain1:%d\n",get_value_ain1);
   
    double pressure = 0.0;
    // 获取压力值的 kpa
    pressure = ((((double)get_value_ain1/1000.0) - 20.0)*10.0)/10.0;
    printf("-------------------------------------------------------------------------------pressure:%f\n",pressure);
    // 获取到最大压力值，打开制冷机
    init_config();
    // 加载配置
    // cJSON* config = get_pressure_voltage_config();
    // if (!config) {
    //     fprintf(stderr, "Failed to load config, using default config\n");
    //     printf("---config is NULL---\n");
    //     return 0;
    // }
    
    // 加载配置
    double max_pressure = get_max_pressure();
    double epsilon = 0.05; // 误差阈值
    
    if (compare_double(pressure, max_pressure, epsilon) == 1) // a 大于 b
    {
        pressure_operation_open_flag = 1;
        //打开制冷机
        handle_modbus_operation(OP_START_COOLER,0,0);
            //  
    }printf("-------------------------------------------------------------------------------open pressure");  
    
    // 获取到最小压力值，关闭制冷机
    double min_pressure = get_min_pressure();
    if (compare_double(pressure, min_pressure, epsilon) == -1) // a 小于 b
    {
        pressure_operation_close_flag = 1;
        handle_modbus_operation(OP_STOP_COOLER,0,0);
            //printf("-------------------------------------------------------------------------------close pressure");
    }

    // 当大于压力值后， 则报警
    double err_pressure = get_err_pressure();
    if (compare_double(pressure, err_pressure, epsilon) == 1) // a 大于 b
    {
        error_pressure_operation_buzzer_flag = 1;
        printf("---error_pressure_operation_buzzer_flag ---\n");
    } else
    {
        error_pressure_operation_buzzer_flag = 0;
    }

    //printf("---pressure: %.1f, max_pressure: %.1f, min_pressure: %.1f, err_pressure: %.1f ---\n", pressure, max_pressure, min_pressure, err_pressure);

    // 释放配置
    // cJSON_Delete(config);
}


double convert_to_kilo_ceil(uint32_t value) {
    // 步骤1：恢复实际的压力值
    // 存储值 = (实际压力 + 20.0) × 1000
    // 所以：实际压力 = (存储值 / 1000.0) - 20.0
    double actual_pressure = ((double)value / 1000.0) - 20.0;
    //pressure = ((((double)get_value_ain1/1000.0) - 20.0)*10.0)/10.0;
    // 步骤2：保留1位小数并向上取整
    double result = ceil(actual_pressure * 10.0) / 10.0;
    
    return result;
}
```

液位显示基本同理，继电器的打开关闭

```

// 根据液位值，判断是否符合开启或者关闭的阈值
/**
 * @brief 液位低于我们设定值，比如说30%，这时候就要往液氮罐里补充液氮。就需要继电器给出一个信号，控制外界的输送装置打开阀门
 * 
 * @param liquid_persent 
 */
void liquid_operation_status_with_value(int liquid_persent) {
    // 加载配置
    // cJSON* config = get_pressure_voltage_config();
    // if (!config) {
    //     fprintf(stderr, "Failed to load config, using default config\n");
    //     printf("---config is NULL---\n");
    //     return 0;
    // }
    
    // 加载配置
    init_config();
    
    printf("-------------------------------------------------------------------------------get_value_ain2:%d\n",get_value_ain2);
    //double liquid_value = (double)liquid_persent;
    //liquid_persent = (double)get_value_ain2/10.0
    double liquid_value = (double)get_value_ain2/10.0;

    // 获取到配置文件的最大和最小值
    double max_liquid = get_max_liquid();
    double epsilon = 1e-6; // 误差阈值
    
    if (compare_double(liquid_value, max_liquid, epsilon) == 1) // a 大于 b
    {
        // 关闭继电器    
        control_gpio26(48);
    }
    // 获取到配置文件的最大和最小值
    double min_liquid = get_min_liquid();
    if (compare_double(liquid_value, min_liquid, epsilon) == -1) // a 小于 b
    {
        // 开启继电器
        control_gpio26(49);
    }

    // 获取到配置文件的错误报警阈值
    double err_liquid = get_err_liquid();
    if (compare_double(liquid_value, err_liquid, epsilon) == 1) // a 大于 b
    {
        error_liquid_operation_buzzer_flag = 1;
        printf("---error_liquid_operation_buzzer_flag ---\n");
    } else
    {
        error_liquid_operation_buzzer_flag = 0;
    }

    printf("---liquid: %.1f, max_liquid: %.1f, min_liquid: %.1f, err_liquid: %.1f ---\n", liquid_value, max_liquid, min_liquid, err_liquid);

    // 释放配置
    // cJSON_Delete(config);
}
```





![image-20251224141926390](ite.assets/image-20251224141926390.png)
