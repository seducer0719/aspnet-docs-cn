
检查 Details 和 Delete 方法
=====================================================

由 `Rick Anderson`_ 编辑

打开 Movie 控制器并检查 ``Details`` 方法:

.. literalinclude:: start-mvc/sample2/src/MvcMovie/Controllers/MoviesController.cs
  :language: c#
  :start-after:  // GET: Movies/Details/5
  :end-before: // GET: Movies/Create
  :dedent: 8

创建这个操作方法的MVC支架式引擎添加了一条注释表明一个HTTP请求会调用此方法，这种情况下，它是由三部分URL组成的一个GET请求 ——  ``Movies`` 控制器，  ``Details`` 方法 和 一个 ``id`` 的值. 回调这些部分被定义在 Startup中。

.. literalinclude:: start-mvc/sample2/src/MvcMovie/Startup.cs
  :dedent: 12
  :emphasize-lines: 6
  :start-after: app.UseIdentity();
  :end-before: SeedData.Initialize(app.ApplicationServices);

代码先行使得利用 ``SingleOrDefaultAsync`` 方法查询数据变得十分简单。一个重要的内置安全功能就是代码会在其本身利用查询方法做什么之前验证查询方法确实返回了一个结果。比如，一个黑客可以通过把创建的URL *http://localhost:xxxx/Movies/Details/1* 更改为类似  *http://localhost:xxxx/Movies/Details/12345* 的URL（或者其他一些不代表真实的电影的值）来把错误导入到网站中。 如果你没有对查询电影的结果进行空值检查，程序就会抛出异常。

检查 ``Delete`` 和 ``DeleteConfirmed`` 方法.

.. literalinclude:: start-mvc/sample2/src/MvcMovie/Controllers/MoviesController.cs
  :language: c#
  :start-after:  // GET: Movies/Delete/5
  :end-before: private bool MovieExists(int id)
  :dedent: 8

注意 ``HTTP GET Delete`` 方法并不会删除指定的电影 doesn't delete the specified movie, 它返回一个可以让你提交(HttpPost)删除电影操作的视图。 执行删除操作对GET请求做出响应（或者执行编辑操作，创建操作，以及其他任何更改数据的操作）会产生一个安全黑洞。

删除数据的 ``[HttpPost]`` 方法 被命名为 ``DeleteConfirmed`` ，给 HTTP POST 方法一个独特的标识或名字。 这两个方法的标识如下所示：

.. code-block:: c#

        // GET: Movies/Delete/5
        public async Task<IActionResult> Delete(int? id)  
  
        // POST: Movies/Delete/
        [HttpPost, ActionName("Delete")]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> DeleteConfirmed(int id)

公共语言运行库 (CLR) 需要重载的方法从而具有一个独特的变量标识（同样的方法但是不同的变量队列）。 然而这里你需要两个具有同样变量标识的 ``Delete`` 方法 —— 一个 GET ，一个 POST。 （它们都需要接收一个整型作为变量）

对于这个问题有两个解决方法，一个是给方法不同的名字。这就是支架机制在前面样例所做的。 然而，这里指出一个小问题： ASP.NET 以名字匹配URL片段和操作方法，如果你重命名一个方法，路由通常无法找到这个方法。解决方法就如你在样例所见，把 ``ActionName("Delete")`` 属性加到 ``DeleteConfirmed`` 方法上。 这个属性为路由系统执行匹配以便包含 /Delete/ 的URL的POST请求可以找到 ``DeleteConfirmed`` 方法。

另外一种常见的避免相同名字方法的解决方案是人为地改变POST方法的标识，在其中添加一个额外（未被使用）的变量。 这就是我们在先前POST中所做的，添加的 ``notUsed`` 变量。 在这里你可以对 ``[HttpPost] Delete`` 方法做同样的事情：

.. literalinclude:: start-mvc/sample2/src/MvcMovie/Controllers/MoviesController.cs
  :language: c#
  :start-after:  // POST: Movies/Delete/6
  :end-before: // End of DeleteConfirmed
  :dedent: 8


.. ToDo - Next steps, but it really needs to start with Tom's EF/MVC Core