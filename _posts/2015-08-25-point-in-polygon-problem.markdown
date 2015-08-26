---
layout: post
title:  "Point in polygon problem"
tags: algorithms computational-geometry
---

Known as [Point-In-Polygon](https://en.wikipedia.org/wiki/Point_in_polygon) problem, testing whether a point lies inside a polygon is another classic literature problem in computational geometry.

As a prerequiste for this post, make sure to read [Line segments intersection]({% post_url 2015-08-24-line-segments-intersection %}).

A simple test to check whether a point \\( p=\\{x,y\\} \\) lies inside a given polygon is to extend one of its dimensions to infinity 

$$ p_{inf} = \{ +\infty,y \} $$

and do a [line segments intersection]({% post_url 2015-08-24-line-segments-intersection %}) test between \\( \overline{p \ p_{inf}} \\) and each one of the edges of the polygon. If the count of the intersections is odd, the point lies inside the polygon.

![image](/images/posts/pointinpolygonproblem1.png)

Special attention deserves the case when segment \\( \overline{p \ p_{inf}} \\) has a successful intersection test and \\( p \\) is found collinear to the polygon edge: if \\( p \\) lies in the polygon edge segment, any further check can be skipped since the point lies on the border of the polygon.

Algorithm follows

{% highlight c++ %}
#include <iostream>
#include <string>
#include <vector>
#include <iterator>
#include <algorithm>
using namespace std;

// NB: max_int would cause overflow in the orientation test
const int INF = 100'000;

struct Point {
  int x, y;
};

ostream& operator<<(ostream& os, const Point& p) {
  os << "{" << p.x << ";" << p.y << "}";
  return os;
}
ostream& operator<<(ostream& os, const vector<Point>& p) {
  os << "{";
  copy(p.begin(), p.end(), ostream_iterator<Point>(os));
  os  << "}";
  return os;
}

int orientation(const Point& p1, const Point& p2, const Point& q1) {
  int val = (p2.y - p1.y) * (q1.x - p2.x) - (q1.y - p2.y) * (p2.x - p1.x);
  if (val == 0)
    return 0;
  else
    return (val < 0) ? -1 : 1;
}

// Returns true if q lies on p1-p2
bool onSegment(const Point& p1, const Point& p2, const Point& q) {
  if (min(p1.x, p2.x) <= q.x && q.x <= max(p1.x, p2.x)
    && min(p1.y, p2.y) <= q.y && q.y <= max(p1.y, p2.y))
    return true;
  else
    return false;
}

bool intersectionTest(const Point& p1, const Point& p2,
  const Point& p3, const Point& p4) {
  int o1 = orientation(p1, p2, p3);
  int o2 = orientation(p1, p2, p4);
  int o3 = orientation(p3, p4, p1);
  int o4 = orientation(p3, p4, p2);

  // General case
  if (o1 != o2 && o3 != o4)
    return true;

  // Special cases
  if (o1 == 0 && onSegment(p1, p2, p3))
    return true;
  if (o2 == 0 && onSegment(p1, p2, p4))
    return true;
  if (o3 == 0 && onSegment(p3, p4, p1))
    return true;
  if (o4 == 0 && onSegment(p3, p4, p2))
    return true;

  return false;
}

bool pointInPolygon(const Point& p, const vector<Point>& polygon) {

  if (polygon.size() < 3)
    return false; // Flawed polygon

  Point PtoInfinity = { INF , p.y };

  int intersectionsCount = 0;
  int i = 0, j = i + 1;
  while (j != 0) {

    if (intersectionTest(p, PtoInfinity, polygon[i], polygon[j]) == true) {

      ++intersectionsCount;

      if (orientation(polygon[i], polygon[j], p) == 0) { // Collinear
        if (onSegment(polygon[i], polygon[j], p) == true)
          return true;
      }
    }

    i = (++i % polygon.size());
    j = (++j % polygon.size());;
  }

  return (intersectionsCount % 2 != 0);
}

int main() {

  auto printPointInPolygon = [](auto p, auto& polygon) {
    cout << boolalpha << "Point " << p << " lies in polygon " << polygon <<
      " - " << pointInPolygon(p, polygon) << endl;
  };

  vector<Point> polygon = { {0,0}, {0,3}, {3,3}, {3,1}, {2,1}, {2,0} };
  Point p = { 1,1 };
  printPointInPolygon(p, polygon);


  polygon = { { 0, 0 },{ 5, 0 },{ 10, 10 },{ 5, 10 } };
  p = { 3, 3 };
  printPointInPolygon(p, polygon);

  p = { 4, 10 };
  printPointInPolygon(p, polygon);

  polygon = { { 0, 0 },{ -5, 0 },{ -10, -10 } };
  p = { 0, -2 };
  printPointInPolygon(p, polygon);

  return 0;
}
{% endhighlight %}


References
==========
- [Geometric algorithms course slides - cs233](http://www.dcs.gla.ac.uk/~pat/52233/slides/Geometry1x1.pdf)
- [Point in polygon problem](http://www.geeksforgeeks.org/how-to-check-if-a-given-point-lies-inside-a-polygon/)