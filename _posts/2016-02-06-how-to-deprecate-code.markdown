---
layout: post
title:  "How to deprecate a codebase"
date:   2016-02-06
---

This is a paragraph.

Then this is.

``` javascript
import Promise from 'bluebird'

export default function (bookshelf) {
  bookshelf.Model.prototype.deferredFetchAll = function (opts = {}) {
    const query = this.query(q => q.select([this.idAttribute])).query().clone()
    this.resetQuery()

    const { sql, bindings } = query.toSQL()

    // TODO: make sure the order is correct, write test
    return this.query(q =>
      q.joinRaw(`inner join (${sql}) as _d using (${this.idAttribute})`, bindings)
    ).fetchAll(opts)
  }
}
```
