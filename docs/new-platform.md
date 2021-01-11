- If the Bazel Rust and Node rules do not support your platform, extra work may be required.
- Anki uses the Python 'orjson' module to speed up DB access. If you're on a
  platform that can not build orjson, you can remove it from
  pip/requirements.txt to skip it during running/building, but DB operations
  will be slower.

  The py_wheel() rule in pylib/anki/BUILD.bazel adds an orjson requirement to
  the generated Anki wheel on x86_64. If you have removed orjson, you'll want to
  remove that line. If you have successfully built orjson for another platform,
  you'll want to adjust that line to include your platform.
If node doesn't provide a binary for your platform and you have a local copy
installed, you can create a local_node folder in the project root, symlink in
your local installation, and modify defs.bzl.

```patch
diff --git a/defs.bzl b/defs.bzl
index eff3d9df2..fb2e9f7fe 100644
--- a/defs.bzl
+++ b/defs.bzl
@@ -41,7 +41,15 @@ def setup_deps():
         python_runtime = "@python//:python",
     )

-    node_repositories(package_json = ["@net_ankiweb_anki//ts:package.json"])
+    native.local_repository(
+        name = "local_node",
+        path = "local_node",
+    )
+
+    node_repositories(
+        package_json = ["@net_ankiweb_anki//ts:package.json"],
+        vendored_node = "@local_node//:node",
+    )

     yarn_install(
         name = "npm",
diff --git a/local_node/BUILD.bazel b/local_node/BUILD.bazel
new file mode 100644
index 000000000..aa0c473ae
--- /dev/null
+++ b/local_node/BUILD.bazel
@@ -0,0 +1 @@
+exports_files(["node/bin/node"] + glob(["node/lib/node_modules/**"]))
diff --git a/local_node/WORKSPACE b/local_node/WORKSPACE
new file mode 100644
index 000000000..e69de29bb
diff --git a/local_node/node/bin/node b/local_node/node/bin/node
new file mode 120000
index 000000000..d7b371472
--- /dev/null
+++ b/local_node/node/bin/node
@@ -0,0 +1 @@
+/usr/local/bin/node
\ No newline at end of file
diff --git a/local_node/node/lib/node_modules b/local_node/node/lib/node_modules
new file mode 120000
index 000000000..23dd0736e
--- /dev/null
+++ b/local_node/node/lib/node_modules
@@ -0,0 +1 @@
+/usr/local/lib/node_modules
\ No newline at end of file
```