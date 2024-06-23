- 1.入门

  - 设计Duck超类，其他各种鸭子继承此超类

  - 问题：想要鸭子能飞

    - 方法一：直接在Duck类中加入fly()方法

      - 问题：会导致所有的鸭子都具备fly()方法

        - 解决：在继承时将不需要的方法直接覆盖

          - 问题：

            - 不会叫的鸭子又要将quack()覆盖

            - 重复累赘，牵一发而动全身

    - 方法二：创建Flyable、Quackable接口，会飞、会叫的鸭子分别实现该接口
      - 问题：接口不具有实现代码，各鸭子的fly()代码无法复用

    - 原则一：找出应用中可能需要变化之处，把它们独立出来，不要和那些不需要变化的代码混在一起
      - 把会变化的部分取出并封装起来，以便以后可以轻易地改动或扩充此部分，而不影响不需要变化的其他部分

    - 方法三：将鸭子的行为从Duck类中独立出来，分别创建FlyBehavior和QuackBehavior接口（算法簇），特定的具体行为（算法）都在实现给接口的类中，委托这些类对鸭子的行为进行处理

      ```java
      public interface FlyBehavior {
      	public void fly();
      }
      public class FlyWithWings implements FlyBehavior {
      	public void fly() {
      		System.out.println("I'm flying!!");
      	}
      }
      public class FlyNoWay implements FlyBehavior {
      	public void fly() {
      		System.out.println("I can't fly");
      	}
      }
      public interface QuackBehavior {
      	public void quack();
      }
      public class Quack implements QuackBehavior {
      	public void quack() {
      		System.out.println("Quack");
      	}
      }
      public class MuteQuack implements QuackBehavior {
      	public void quack() {
      		System.out.println("<< Slience >>");
      	}
      }
      public class Squeak implements QuackBehavior {
      	public void quack() {
      		System.out.println("Squeak");
      	}
      }
      public class Duck{
      	FlyBehavior flyBehavior;
      	QuackBehavior quackBehavior;
      	
      	public Duck() {
      		
      	}
      	
      	public void setFlyBehavior(FlyBehavior fb) { // 动态设定
      		flyBehavior = fb;
      	}
      	
      	public void setQuackBehavior(QuackBehavior qb) {
      		quackBehavior = qb;
      	}
      	
      	public void performQuack() {
      		quackBehavior.quack();
      	}
      	
      	public void performFly() {
      		flyBehavior.fly();
      	}
      }
      public class MallardDuck extends Duck {
      	public MallardDuck() {
      		quackBehavior = new Quack();
      		flyBehavior = new FlyWithWings();
      	}
      	
      	public void display() {
      		System.out.println("I'm a real Mallard duck");
      }
      public class MiniDuckSimulator {
      	public static void main(String() args) {
      		Duck mallard = new MallardDuck();
      		mallard.performQuack();
      		mallard.performFly();
      	}
      }	
      ```

    - 原则二：针对接口（超类）编程，而非针对实现编程

      - 例如：

        - 针对实现：

          - Dog d = new Dog();

          - d.bark();

        - 针对接口/超类型：

          - Animal animal = new Dog();

          - animal.makeSound();

        - 运行时确定具体对象：

          - a = getAnimal();

          - a.makeSound();

    - 原则三：多用组合，少用继承

  - 策略模式（Strategy Pattern）：定义算法簇将各类算法分别封装，使它们之间可以相互替换，从而让算法的变化独立于使用算法的客户

- 2. 观察者模式

  - 设计WeatherDate类，可以用该类中的方法获取及更新气象数据

  - 问题：要调用measurementsChanged()方法，获取新数据的同时更新几个布告板上的信息

    - 方法一：获取数据后直接依次更新

      ```java
      public class WeatherData {
          public void measurementsChanged() {
              float temp = getTemperature();
              float humidity = getHumidity();
              float pressure = getPressure();
              
              currentCoditionsDisplay.update(temp, humidity, pressure);
              statisticsDisplay.update(temp, humidity, pressure);
              forecastDisplay.update(temp, humidity, pressure);
          }
      }
      ```

      - 问题：针对具体实现编程，布告板的可扩展性差

    - 方法二：采用观察者模式

      ```java
      public interface Subject {
          public void registerObserver(Observer o);
          public void removeObserver(Observer o);
          public void notifyObserver();
      }
      public interface Observer {
          public void update(float temp, float humidity, float pressure);
      }
      public interface DisplayElement {
          public void display();
      }
      ​
      public class WeatherData implements Subject {
          private ArrayList observers;
          private float temperature;
          private float humidity;
          private float pressure;
          
          public WeatherData() {
              observers = new ArrayList();
          }
          
          public void registerObserver(Observer o) {
              observers.add(o);
          }
          
          public void removeObserver(observer o) {
              int i = observer.indexof(o);
              if (i >= 0)
                  observers.remove(i);
          }
          
          public void notifyObservers() {
              for (int i = 0; i < observers.size(); i++) {
                  Observer observer = (Observer)observers.get(i);
                  observer.update(temperature, humidity, pressure);
              }
          }
          
          public void measurementsChanged() {
              notifyObservers();
          }
          
          public void setMeasurements(float temperature, float humidity, float pressure) {
              this.temperature = temperature;
              this.humidity = humidity;
              this.pressure = pressure;
              measurementsChanged();
          }
      }
      ​
      public class CurrentConditionsDisplay implements Observer, DisplayElement {
          private float temperature;
          private float humidity;
          private Subject weatherData;
          
          public CurrentConditionsDisplay(Subject weatherData) {
              this.weatherData = weatherData;
              weatherData.registerObserver(this);
          }
          
          public void update(float temperature, float humidity, float pressure) {
              this.temperature = temperature;
              this.humidity = humidity;
              display();
          }
          
          public void display() {
              System.out.println("Current condition:" + temperature + "F degree and " + humidity + "% humidity");
          }
      }
      ​
      public class WeatherStation {
          public static void main(String[] args) {
              WeatherData weatherData = new WeatherData();
              
              CurrentConditionsDisplay currentDisplay = new CurrentConditionsDisplay(weatherData);
              StatisticsDisplay statisticsDisplay = new StatisticsDisplay(weatherData);
              ForecastDisplay forecastDisplay = new ForecastDisplay(weatherData);
              
              weatherData.setMeasurements(80, 65, 30.4f);
          }
      }
      ```

      - Java内置了观察者模式，java.util包中包含最基本的Observer和Observable类，可以用推（push）或拉（pull）的方式传送数据

        ```java
        import java.util.Observable;
        
        public class WeatherData extends Observable {
            private float temperature;
            private float humidity;
            private float pressure;
            public WeatherData() {
            }
            public void measurementsChanged() {
                setChanged();
                notifyObservers();
            }
            public void setMeasurements(float temperature, float humidity, float pressure) {
                this.temperature = temperature;
                this.humidity = humidity;
                this.pressure = pressure;
                measurementsChanged();
            }
            public float getTemperature() {
                return temperature;
            }
            public float getHumidity() {
                return humidity;
            }
            public float getPressure() {
                return pressure;
            }
        }
        public interface DisplayElement {
            public void display();
        }
        ​
        import java.util.Observable;
        import java.util.Observer;
        ​
        public class CurrentConditionsDisplay implements Observer, DisplayElement {
            private Observable observable;
            private float temperature;
            private float humidity;
            public CurrentConditionsDisplay(Observable observable) {
                this.observable = observable;
                observable.addObserver(this);
            }
            public void update(Observable obs, Object arg) {
                if (obs instanceof WeatherData) {
                    WeatherData weatherData = (WeatherData)obs;
                    this.temperature = weatherData.getTemperature();
                    this.humidity = weatherData.getHumidity();
                    display();
                }
            }
            public void display() {
                System.out.println("Current conditions: " + temperature + "F degree and " + humidity + "% humidity");
            }
        }
        ```

    - 原则四：为了交互对象之间的松耦合设计而努力

  - 观察者模式（Observer Pattern）：定义对象之间一对多的依赖关系，当一个对象的状态改变时，其所有依赖者都会收到通知并自动更新

    - 主题（可观察者）用共同的接口来更新观察者

    - 观察者和可观察者之间以松耦合（loosecoupling）的方式结合，可观察者不知道观察值的细节，只知道观察值实现了观察值的接口

- 3.装饰者模式

  - 设计Beverage类，各种饮料继承该类而组成订单系统

    - 方法一：简单地继承

      

      - 问题：

        - 添加新调料时需加入新方法并改变cost()

        - 某些调料可能对某些饮料不适用

      - 原则五：类应该对扩展开发，对修改关闭

    - 方法二：装饰者模式

      ```java
      public abstract class Beverage {
      	String description = "Unknown Beveeage";
      	
      	public String getDescription() {
      		return description;
      	}
      	
      	public abstract double cost();
      }
      ​
      public abstract class CondimentDecorator extends Beverage {
      	public abstract String getDescription();
      }
      ​
      public class Espresso extends Beverage {
      	public Espresso() {
      		description = "Esprasso";
      	}
      	
      	public double cost() {
      		return 1.99;
      	}
      }
      public class HouseBlend extends Beverage {
      	public HouseBlend() {
      		description = "House Blend Coffee";
      	}
      	
      	public double cost() {
      		return .89;
      	}
      }
      ​
      public class Mocha extends CondimentDecorator {
      	Beverage beverage;
      	
      	public Mocha(Beverage beverage) {
      		this.beverage = beverage;
      	}
      	
      	public String getDescription() {
      		return beverage.getDescription() + ", Mocha";
      	}
      	
      	public double cost() {
      		return .20 + beverage.cost();
      	}
      }
      ​
      public class StarbuzzCoffee {
      	public static void main(String args[]) {
      		Beverage beverage = new Espresso();
      		System.out.println(beverage.getDescription() + " $" + beverage.const());
      		
      		Beverage beverage2 = new DarkRoaster();
      		beverage2 = new Mocha(beverage2);
      		beverage2 = now Macha(beverage2);
      		System.out.println(beverage2.getDescription() + " $" + beverage2.const());
      	}
      }	
      ```

  - 装饰者模式（Decorator Pattern）：动态地将责任附加到对象上，若要扩展功能，装饰者提供了比继承更有弹性的替代方案
    - 利用继承实现“类型匹配”而非获得“行为”

  