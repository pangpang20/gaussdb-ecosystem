---
name: 🐛 Bug Report
about: If you find something not work fine with GaussDB 🤔.
---

# Open Source Software Info

* Name: e.g Hiberante
* Official Source Code: e.g. https://github.com/hibernate/hibernate-orm/
* Version: e.g. 7.1.0

# You Solution and Problem


e.g.

* Create a proceduer

```
create or replace procedure proc_in_out(a in number, b in number, o out number)
is
begin
    o:=a;
end;
/
```

* Execute by Hibernate

```
public String test002() {
		ProcedureCall query = getCurrentSession().createStoredProcedureCall("proc_in_out");
		query.registerStoredProcedureParameter("b", Integer.class, ParameterMode.IN)
				.setParameter("b", 2);
		query.registerStoredProcedureParameter("a", Integer.class, ParameterMode.IN)
				.setParameter("a", 1);
		query.registerStoredProcedureParameter("o", Integer.class, ParameterMode.OUT);
		query.execute();
	}
```

* The Problem

Expect successfully executed but got an error. Details info ...