## 摘要

在不破坏封装性的前提下，捕获一个对象的内部状 态，并在该对象之外保存这个状态。这样以后就可将该对象恢复到原先保存的状态。

## 类图

以例子说明, 一个游戏角色打boss, 打的过程中可以保存角色状态: 血量攻击防御

![image-20201024193659263](https://gitee.com/p8t/picbed/raw/master/imgs/20201024193700.png)

## 实现

```java
class Role {
    private int vitality = 100;
    private int attack = 100;
    private int defence = 100;

    private final char[] c = {'!', '@', '#', '$', '%', '^', '&', '*', ',', '/', '"', '~'};

    public SavePoint createSavePoint() {
        return new SavePoint(vitality, attack, defence);
    }

    public void restoreSavePoint(SavePoint savePoint) {
        this.vitality = savePoint.getVitality();
        this.attack = savePoint.getAttack();
        this.defence = savePoint.getDefence();
    }

    public void fight() {
        Random r = new Random();
        System.out.println("========fight========");
        StringBuilder s = new StringBuilder();
        for (int i = 0; i < 20; i++) {
            s.append(c[r.nextInt(12)]);
        }
        System.out.print(s + "\n");
        this.vitality -= r.nextInt(10) + 10;
        this.attack -= r.nextInt(10) + 1;
        this.defence -= r.nextInt(5) + 1;
    }

    public void showStatue() {
        System.out.println("========status========");
        System.out.println("vitality: " + vitality);
        System.out.println("attack: " + attack);
        System.out.println("defence: " + defence);
    }
}
```

```java
// 对角色状态的封装, 可以决定记录哪些状态, 怎样进行记录
class SavePoint {
    private int vitality;
    private int attack;
    private int defence;

    public SavePoint(int vitality, int attack, int defence) {
        this.vitality = vitality;
        this.attack = attack;
        this.defence = defence;
    }

    public int getVitality() {
        return vitality;
    }

    public int getAttack() {
        return attack;
    }

    public int getDefence() {
        return defence;
    }
}
```

```java
// 用来管理角色状态的
public class Caretaker {
    private SavePoint savePoint;

    public SavePoint getSavePoint() {
        return savePoint;
    }

    public void setSavePoint(SavePoint savePoint) {
        this.savePoint = savePoint;
    }
}
```

## 测试

```java
public static void main(String[] args) {
    Role role = new Role();
    Caretaker caretaker = new Caretaker();

    role.fight();
    role.showStatue();
    role.fight();
    role.showStatue();

    caretaker.setSavePoint(role.createSavePoint());
    System.out.println("create savepoint");

    role.fight();
    role.showStatue();
    role.fight();
    role.showStatue();

    System.out.println("restore savepoint");

    role.restoreSavePoint(caretaker.getSavePoint());
    role.showStatue();
}
/*
    ========fight========
    $""@*#%~$^#%@~~^~%*,
    ========status========
    vitality: 83
    attack: 92
    defence: 96
    ========fight========
    !/&,^~,&*!"&/~*^#*%"
    ========status========
    vitality: 68
    attack: 91
    defence: 94
    create savepoint
    ========fight========
    *~%^%@/"**~/,!,&@$&,
    ========status========
    vitality: 52
    attack: 84
    defence: 90
    ========fight========
    *,",,$$$#%~&"#&,%,~!
    ========status========
    vitality: 34
    attack: 83
    defence: 86
    restore savepoint
    ========status========
    vitality: 68
    attack: 91
    defence: 94
*/
```

