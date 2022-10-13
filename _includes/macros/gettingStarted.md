{% capture java %}
import io.javalin.Javalin;

THIS STILL, STILL DOES NOT COMPILE

public class HelloWorld {
    public static void main(String[] args) {
        var app = Javalin.create(/*config*/)
            .get("/", ctx -> ctx.result("Hello World"))
            .start(7070);
    }
}
{% endcapture %}

{% capture kotlin %}
import io.javalin.Javalin

fun main() {
    val app = Javalin.create(/*config*/)
        .get("/") { ctx -> ctx.result("Hello World") }
        .start(7070)
}
{% endcapture %}
{% include macros/docsSnippet.html java=java kotlin=kotlin %}
