# Android 最佳实践

## 建造者模式

当你有一个需要超过3个构造参数的对象时，使用一个构建器来构造这个对象,编写起来可能稍微冗长一点，但是它可以更好地扩展，而且可读性更强。如果您正在创建一个值类，请考虑[AutoValue](https://medium.com/rocknnull/no-more-value-classes-boilerplate-the-power-of-autovalue-bbaf36cf8bbe#.cazel3w3g)。

```java
class Movie {
    static class Builder {
        String title;
        Builder withTitle(String title) {
            this.title = title;
            return this;
        }
        Movie build() {
            return new Movie(title);
        }
    }

    private Movie(String title) {
    [...]    
    }
}
// Use like this:
Movie matrix = new Movie.Builder().withTitle("The Matrix").build();
```

## 静态工厂方法

使用静态工厂方法（和一个私有的构造函数）来代替用 new关键字和构造函数。这些工厂方法可以被命名，使用不同的方法名字使得其构造的对象更加明晰，不需要每次返回对象的新实例，并且可以根据需要返回不同的子类型。

```java
class Movie {
    [...]
    public static Movie create(String title) {
        return new Movie(title);
    }
}
```

## 静态内部类

如果您定义的内部类不依赖于外部类，请不要忘记将其定义为静态。不这样做将导致内部类的每个实例都有对外部类的引用。

```java
class Movie {
    [...]
    static class MovieAward {
        [...]
    }
}
```

## 返回空集合

当不得不返回一个没有结果的列表/集合时，请避免使用null。返回一个空集合可以使接口更简单（不需要为返回空的函数记录和注释），并避免意外的**NPE**。最好返回相同的空集合，而不是创建一个新集合。

```java
List<Movie> latestMovies() {
    if (db.query().isEmpty()) {
        return Collections.emptyList();
    }
    [...]
}
```

## 使用StringBuilder

如果不得不连接几个字符串，`+`运算符可以实现。但切勿使用它进行大量的字符串连接; 性能很差。进行大量的字符串连接可以使用StringBuilder。

```java
String latestMovieOneLiner(List<Movie> movies) {
    StringBuilder sb = new StringBuilder();
    for (Movie movie : movies) {
        sb.append(movie);
    }
    return sb.toString();
}
```

## 强制不能实例化

如果您不想使用**new**关键字创建对象，可以使用**private构造函数**强制执行。对于仅包含静态函数的工具类特别有用。

```java
class MovieUtils {
    private MovieUtils() {}
    static String titleAndYear(Movie movie) {
        [...]
    }
}
```

## 避免可变性

不可变是一个在整个生命周期中保持不变的对象。对象的所有必要数据都在创建时提供。这种方法有很多优点，如简单，线程安全和可共享。

```java
class Movie {
    [...]
    Movie sequel() {
        return Movie.create(this.title + " 2");
    }
}
// Use like this:
Movie toyStory = Movie.create("Toy Story");
Movie toyStory2 = toyStory.sequel();
```

使每个类不可变是很困难的。如果的确需要这样的处理，让你的类尽可能不变（例如，private final 字段和final 类）。在移动设备上创建对象可能会更昂贵，因此不要过度使用它。