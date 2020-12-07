Comparable 和 Comparator是Java核心API提供的两个接口，从它们的名字中，我们大致可以猜到它们用来做对象之间的比较的。但它们到底怎么用，它们之间有又哪些差别呢？下面有两个例子可以很好的回答这个问题。下面的例子用来比较HDTV的大小。看完下面的代码，相信对于如何使用Comparable和Comparator会有一个更加清晰的认识。

### Comparable
一个实现Comparable接口的类，可以让自身的对象和其他对象进行比较。也就是说，同一个类的两个对象之间要想比较，对应的类 就要实现Comparable接口，并实现compareTo( )方法，代码如下：

```
public class HDTV implements Comparable<HDTV>{

    private int size;

    private String brand;

    public HDTV(int size, String brand) {
        this.size = size;
        this.brand = brand;
    }

    public int getSize() {
        return size;
    }

    public void setSize(int size) {
        this.size = size;
    }

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public int compareTo(HDTV tv) {
        if(this.getSize() > tv.getSize()){
            return 1;
        }else if(this.getSize() < tv.getSize()){
            return -1;
        }else {
            return 0;
        }
    }

    public static void main(String[] args) {
        HDTV samsung = new HDTV(55, "Samsung");
        HDTV sony = new HDTV(60, "Sony");

        if(samsung.compareTo(sony)>0){
            System.out.println(samsung.getBrand() + " is better.");
        }else {
            System.out.println(sony.getBrand() + " is better.");
        }
    }
}
```

### Comparator
在一些情况下，不希望修改一个原有的类，但是还想让他可以比较，Comparator接口可以实现这样的功能。通过使用Comparator接口，你可以针对其中特定的属性/字段来进行比较。比如，当我们要比较两个人的时候，我可能通过年龄比较、也可能通过身高比较。这种情况使用Comparable就无法实现（因为要实现Comparable接口，其中的compareTo方法只能有一个，无法实现多种比较）。

通过实现Comparator接口同样要重写一个方法：compare()。接下来的例子就通过这种方式来比较HDTV的大小。其实Comparator通常用于排序。Java中的Collections和Arrays中都包含排序的sort方法，该方法可以接收一个Comparator的实例（比较器）来进行排序。

```
public class HDTV {

    private int size;

    private String brand;

    public HDTV(int size, String brand) {
        this.size = size;
        this.brand = brand;
    }

    public int getSize() {
        return size;
    }

    public void setSize(int size) {
        this.size = size;
    }

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public static void main(String[] args) {
        HDTV samsung = new HDTV(55, "Samsung");
        HDTV sony = new HDTV(60, "Sony");
        HDTV panda = new HDTV(42, "Panda");
        List<HDTV> hdtvs = new ArrayList();
        hdtvs.add(samsung);
        hdtvs.add(sony);
        hdtvs.add(panda);

        Collections.sort(hdtvs, new SizeComparator());
        for (HDTV hdtv : hdtvs) {
            System.out.println(hdtv.getBrand());
        }
    }
}

class SizeComparator implements Comparator<HDTV> {

    public int compare(HDTV o1, HDTV o2) {
        if (o1.getSize() > o2.getSize()) {
            return 1;
        } else if (o1.getSize() < o2.getSize()) {
            return -1;
        } else {
            return 0;
        }
    }
}
```
