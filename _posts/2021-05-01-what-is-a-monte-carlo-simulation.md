---
usemathjax: true
excerpt: A monte carlo simulation is a method of estimating an unknown result by continuous random sampling
---

# Motivation

<p style="text-align: center">
	<img src="/assets/images/square.png" />
</p>

Given a square whose sides have a length of $$\mathbf{x}$$ each, the area of such a square is $$\mathbf{x ^ 2}$$. Now suppose we draw a line on one of its diagonals such that we have two right-angled triangle. To find the area of one of triangles, it is $$\mathbf{half \times base \times height}$$. In this case, both the base and height of the triangle are $$\mathbf x$$ so it becomes $$\mathbf{half \times x ^ 2}$$. From this, we can say that $$\mathbf{area of the triangle = half \times area of the square}$$ (since area of the square is $$\mathbf{x ^ 2}$$). Finally, the equation can be rewritten as $$\mathbf{area of the triangle \div area of the square = half}$$. The value of half is $$\mathbf{\frac 1 2}$$ or $$\mathbf{0.5}$$ but for the sake of illustration let us pretend we do not know what the value of half is. The formula tells us that the ratio of the area of the triangle to the area of the square should give us the value of half. With this knowledge, we can fill the sqaure with alot of points and find the ratio of the number of points that landed in the triangle to the number of total points in the square. That should give us an approximation of half. The video below illustrates the concept above:

<video width="640" height="480" controls autoplay>
	<source src="/assets/videos/output.mp4" type="video/mp4">
</video>


# Monte Carlo Simulation
What we just saw above is known as a monte carlo simulation. The code that generated the above  simulation can be found [here](https://github.com/AmosAidoo/monte_carlo_half\) A monte carlo simulation is a method of estimating an unknown result by continuous random sampling. This is a very important concept which has may applications in the fields of Engineering, Computer Science, Finance, e.t.c. Visit this [page](https://en.wikipedia.org/wiki/Monte_Carlo_method#Applications) to read about the applications of monte carlo simulations.

### References

1. [Monte Carlo Method, Wikipedia](https://en.wikipedia.org/wiki/Monte_Carlo_method)
2. [LeiosOS](https://www.youtube.com/watch?v=AyBNnkYrSWY)