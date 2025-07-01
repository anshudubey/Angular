# Angular
new line

 Angular 17 automatically removes component-associated styles from the DOM when the component is destroyed. In Angular 15, these styles persisted, which some applications might have relied on implicitly.

**************************************************************************
public class RequestContext {
    private static final ThreadLocal<String> idHolder = new ThreadLocal<>();

    public static void setId(String id) {
        idHolder.set(id);
    }

    public static String getId() {
        return idHolder.get();
    }

    public static void clear() {
        idHolder.remove();
    }
}


@Service
public class MyService {

    public void mainFlow(String id) {
        try {
            RequestContext.setId(id);

            step1();  // no id param
            step2();  // no id param
            saveToDatabase();  // also uses the context

        } finally {
            RequestContext.clear();  // ⚠️ very important to prevent memory leaks
        }
    }

    public void step1() {
        String id = RequestContext.getId();
        System.out.println("Step1: ID = " + id);
    }

    public void step2() {
        String id = RequestContext.getId();
        System.out.println("Step2: ID = " + id);
    }
}

