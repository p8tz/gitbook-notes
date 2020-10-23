## 摘要

定义对象间一种一对多的依赖关系， 使得每当一个对象改变状态，则所有依赖于它的对象都会得到通知并被自动更新

## 类图

![image-20201021170622853](https://gitee.com/p8t/picbed/raw/master/imgs/20201021170624.png)

## 实现

天气变化, 立马通知

```java
public interface IObserver {
    void update(Weather weather);
}

public class Observer implements IObserver {

    @Override
    public void update(Weather weather) {
        System.out.println("current temperature: " + weather.getTemperature());
        System.out.println("current pressure: " + weather.getPressure());
        System.out.println("current humidity: " + weather.getHumidity());
    }
}
```

```java
public interface IWeather {
    void registerObserver(IObserver o);

    void removeObserver(IObserver o);

    void notifyObservers();
}

public class Weather implements IWeather {

    List<IObserver> observers = new ArrayList<>();

    private float temperature;
    private float pressure;
    private float humidity;

    public Weather() {
    }

    public void setWeatherData(float temperature, float pressure, float humidity) {
        this.temperature = temperature;
        this.pressure = pressure;
        this.humidity = humidity;
        notifyObservers();
    }

    @Override
    public void registerObserver(IObserver o) {
        observers.add(o);
    }

    @Override
    public void removeObserver(IObserver o) {
        observers.remove(o);
    }

    @Override
    public void notifyObservers() {
        for (IObserver observer : observers) {
            observer.update(this);
        }
    }

    public float getTemperature() {
        return temperature;
    }

    public float getPressure() {
        return pressure;
    }

    public float getHumidity() {
        return humidity;
    }
}
```

## 测试

```java
public static void main(String[] args) {
    Weather weather = new Weather();
    weather.registerObserver(new Observer());
    weather.setWeatherData(25.9f, 101000.7f, 78.5f);
    weather.setWeatherData(14.5f, 101340.6f, 53.2f);
}
/*
    current temperature: 25.9
    current pressure: 101000.7
    current humidity: 78.5
    current temperature: 14.5
    current pressure: 101340.6
    current humidity: 53.2
*/
```

