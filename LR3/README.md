# Лабораторная работа №3. Потоковая обработка в Apache Flink

# Задание
Выполнить следующие задания из набора заданий репозитория https://github.com/ververica/flink-training-exercises:

+ RideCleanisingExercise
+ RidesAndFaresExercise
+ HourlyTipsExerxise
+ ExpiringStateExercise

# ExerciseBase
Настройка путей к файлам в исходном классе [ExerciseBase](https://github.com/ververica/flink-training-exercises/blob/master/src/main/java/com/ververica/flinktraining/exercises/datastream_java/utils/ExerciseBase.java)

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/90cb7074-4329-45df-bc1e-a87450ae0d95)
# 1. RideCleansingExercise
Решение: в исходном классе [RideCleansigExercise](https://github.com/ververica/flink-training-exercises/blob/master/src/main/java/com/ververica/flinktraining/exercises/datastream_java/basics/RideCleansingExercise.java) был переписан следующий класс.
Фильтр переопределен возвращать true, если начальные и конечные координаты в пределах границ NYC.
Запуск теста:

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/6026925d-cb3a-4d4f-abab-a337af9df460)

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/2bce506f-2c6e-4560-8997-88e8c582cff8)
# 2. RidesAndFaresExercise
Решение: в исходном классе [RidesAndFaresExercies](https://github.com/ververica/flink-training-exercises/blob/master/src/main/java/com/ververica/flinktraining/exercises/datastream_java/state/RidesAndFaresExercise.java) был переписан класс `EnrichmentFunction` следующим образом:

Метод `open` инициализирует 2 переменные для сохранения текущих TaxiRide и TaxiFare

Метод `flatMap1`: событие TaxiRide сохраняется в rideState. Если соответствующий TaxiFare в fareState, то генерируется новый tuple и fareState очищается.

Метод `flatMap2`: событие TaxiFare сохраняется в fareState. Еслли соответсвующий TaxiRide в rideStae, то генерируется новый tuple и rideState сбрасывается.

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/1ce96572-8a9f-44d2-a7a9-a3958b702a3f)

Запуск теста:

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/2500fc3e-80b4-4844-b0b2-b3b680b29159)
# 3. HourlyTipsExercice
Решение: в исходном классе [HourlyTipsExercice](https://github.com/ververica/flink-training-exercises/blob/master/src/main/java/com/ververica/flinktraining/exercises/datastream_java/windows/HourlyTipsExercise.java) был изменён метод `main` и переписан класс `AddTips` следующим образом:

Группируем по id водителя TaxiFare, далее группируем по каждому часу для каждого водителя. Считаем в process с помощью метода AddTips заработанное водителем за каждый час.
Перегруппируем через timeWindowAll и для каждого часа находим максимальный заработок

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/937a7942-f798-474b-b3f3-ba2fd0a4181e)

Запуск теста:

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/4f0d4d16-af35-4799-af63-4da031a1903c)
# 4. ExpiringStateExercise
Решение: в исходном классе [ExpirintStateExercise](https://github.com/ververica/flink-training-exercises/blob/master/src/main/java/com/ververica/flinktraining/exercises/datastream_java/process/ExpiringStateExercise.java) был переписан класс `EnrichmentFunction` следующим образом:

Создаются Valuestate для сохранения актуальных TaxiRide и TaxiFire и инициализируются в методе `open`

Метод `processElement1`:
Если соотвествующий ride у TaxiFire существует, то создаётся кортеж, TaxiFire очищается.
Если TaxiFire не найден, то TaxiRide сохранится в rideState и регистрируется таймер. Таймер нужен для очистки состояния, если не будет найден TaxiFire

Метод `processElement2`:
Если соответствующий fare у TaxiRide существует, то создаётся кортеж, TaxiRide очищается.
Если TaxiRide не найден, то TaxiFare сохранится в fareState и регистрируется таймер. Таймер нужен для очистки состояния, если не будет найден TaxiRide.

Метод `onTimer`:
Метод срабатывает при истечении таймера в processElement1/processElement2, проверяет rideState и fareState, если нет несогласованных событий, они передаются в unmatchedFares или unmatchedRides и состояние очищается.

```
public static class EnrichmentFunction extends KeyedCoProcessFunction<Long, TaxiRide, TaxiFare, Tuple2<TaxiRide, TaxiFare>> {

   // keyed, managed state
   private ValueState<TaxiRide> rideState;
   private ValueState<TaxiFare> fareState;

   @Override
   public void open(Configuration config) {
      rideState = getRuntimeContext().getState(new ValueStateDescriptor<>("saved ride", TaxiRide.class));
      fareState = getRuntimeContext().getState(new ValueStateDescriptor<>("saved fare", TaxiFare.class));
   }

   @Override
   public void processElement1(TaxiRide ride, Context context, Collector<Tuple2<TaxiRide, TaxiFare>> out) throws Exception {
      TaxiFare fare = fareState.value();
      if (fare != null) {
         fareState.clear();
         context.timerService().deleteEventTimeTimer(fare.getEventTime());
         out.collect(new Tuple2(ride, fare));
      } else {
         rideState.update(ride);
         // as soon as the watermark arrives, we can stop waiting for the corresponding fare
         context.timerService().registerEventTimeTimer(ride.getEventTime());
      }
   }

   @Override
   public void processElement2(TaxiFare fare, Context context, Collector<Tuple2<TaxiRide, TaxiFare>> out) throws Exception {
      TaxiRide ride = rideState.value();
      if (ride != null) {
         rideState.clear();
         context.timerService().deleteEventTimeTimer(ride.getEventTime());
         out.collect(new Tuple2(ride, fare));
      } else {
         fareState.update(fare);
         // as soon as the watermark arrives, we can stop waiting for the corresponding ride
         context.timerService().registerEventTimeTimer(fare.getEventTime());
      }
   }

   @Override
   public void onTimer(long timestamp, OnTimerContext ctx, Collector<Tuple2<TaxiRide, TaxiFare>> out) throws Exception {
      if (fareState.value() != null) {
         ctx.output(unmatchedFares, fareState.value());
         fareState.clear();
      }
      if (rideState.value() != null) {
         ctx.output(unmatchedRides, rideState.value());
         rideState.clear();
      }
   }
}

```

Запуск теста: 
![image](https://github.com/badasqi/BigData_LABs/assets/78803025/650818ce-cba6-4cd1-9ca5-58118295156e)




