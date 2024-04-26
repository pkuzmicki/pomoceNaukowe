# oop24
https://github.com/kdmitruk
https://github.com/lukaszkurantdev
https://github.com/michaldziuba03/java

public class Point {
    public final double x;
    public final double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
}

import java.util.List;

public class Polygon {
    private final List<Point> points;

    public Polygon(List<Point> points) {
        this.points = points;
    }

    public boolean inside(Point point) {
        int counter = 0;
        for (int i = 0; i < points.size(); i++) {
            Point pa = points.get(i);
            Point pb = points.get((i + 1) % points.size());
            if (pa.y > pb.y) {
                Point temp = pa;
                pa = pb;
                pb = temp;
            }
            if (pa.y < point.y && point.y < pb.y) {
                double d = pb.x - pa.x;
                double x;
                if (d == 0) {
                    x = pa.x;
                } else {
                    double a = (pb.y - pa.y) / d;
                    double b = pa.y - a * pa.x;
                    x = (point.y - b) / a;
                }
                if (x < point.x) {
                    counter++;
                }
            }
        }
        return counter % 2 != 0;
    }
}


// Kroki 4: Testy jednostkowe dla klasy Polygon
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class PolygonTest {

    // Test sprawdzający czy punkt leży wewnątrz wielokąta
    @Test
    public void testPointInsidePolygon() {
        // Wielokąt o wierzchołkach (1,1), (3,1), (3,3), (1,3)
        Point p1 = new Point(1, 1);
        Point p2 = new Point(3, 1);
        Point p3 = new Point(3, 3);
        Point p4 = new Point(1, 3);
        Polygon polygon = new Polygon(List.of(p1, p2, p3, p4));

        // Sprawdź punkt wewnątrz wielokąta
        Point testPointInside = new Point(2, 2);
        assertTrue(polygon.inside(testPointInside));
    }

    // Test sprawdzający czy punkt leży na zewnątrz wielokąta
    @Test
    public void testPointOutsidePolygon() {
        // Wielokąt o wierzchołkach (1,1), (3,1), (3,3), (1,3)
        Point p1 = new Point(1, 1);
        Point p2 = new Point(3, 1);
        Point p3 = new Point(3, 3);
        Point p4 = new Point(1, 3);
        Polygon polygon = new Polygon(List.of(p1, p2, p3, p4));

        // Sprawdź punkt na zewnątrz wielokąta
        Point testPointOutside = new Point(4, 2);
        assertFalse(polygon.inside(testPointOutside));
    }

    // Test sprawdzający czy punkt leży na prawo od wielokąta
    @Test
    public void testPointOnRightOfPolygon() {
        // Wielokąt o wierzchołkach (1,1), (3,1), (3,3), (1,3)
        Point p1 = new Point(1, 1);
        Point p2 = new Point(3, 1);
        Point p3 = new Point(3, 3);
        Point p4 = new Point(1, 3);
        Polygon polygon = new Polygon(List.of(p1, p2, p3, p4));

        // Sprawdź punkt na prawo od wielokąta
        Point testPointOnRight = new Point(4, 2);
        assertFalse(polygon.inside(testPointOnRight));
    }
}
