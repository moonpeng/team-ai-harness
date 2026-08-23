# CodeGraph

When analyzing repository structure, dependencies, call chains,
or impact of changes, prefer CodeGraph.

Use:

codegraph explore "\<question>"

Examples:

codegraph explore "How does authentication work?"

codegraph explore "How does OrderController reach the database?"

codegraph explore "What depends on PaymentService?"

Avoid repeatedly using grep/find/read when CodeGraph already
provides the required structural information.
