---
layout: post
title:  "Inyección de dependencia"
date:   2019-04-06 21:30:29 +0100
categories: Entorno de desarrollo
---

## Inyección de dependencia  
---  

Este término viene a usarse más comunmente desde la aparición de frameworks como Spring.

Con la inyección de dependencias conseguimos que nuestras piezas de software sean independientes y se comuniquen mediante un interface. Esto nos lleva a suprimir instanciaciones de objetos con la instrucción new.

En este ejemplo **A** debe inicializar la dependencia **b**:
~~~java
class  A {
	private B b;
	public A() {
		b = new B();
	}
}
~~~

En cambio aquí, tenemos inyección de dependencia, **A** deja de ser la responsable de la inicialización y es ahora responsable quien crea A de informar al objeto **b**.

~~~java
public class A {
	private B b;
	public A(B b) {
		this.b = b;
	}
}
~~~

También la podemos encontrar en el uso de los `seter` :
~~~java
public class A {
	private B b;

	...

	public void bSet(B b) {
		this.b = b;
	}
}
~~~

Al buscar esa independencia de cada pieza de código, mediante el uso de interfaces, podemos modificar una pieza de software por otro sin necesidad de reprogramar las clases que hacen uso de él.

Del código OtroProyDI:

`DAOCategory.java`
~~~java
package productservlet;
import java.util.List;

public interface DAOCategory{
  public List<String> listaNombreCat();
  public boolean valida(String cat);
}
~~~

`DAOCategoryImplFile.java`
~~~java
package productservlet;

import java.util.ArrayList;
import java.util.List;
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.File;
import java.io.FileReader;
import java.io.BufferedReader;
import java.io.IOException;

import javax.enterprise.inject.Alternative;

@Alternative
public class DAOCategoryImplFile implements DAOCategory{
  public List<String> listaNombreCat() {
     List<String> result = new ArrayList<String>();
     FileReader fileReader = null;
     try {
      File file = new File("/tmp/CategoryFile");
      fileReader = new FileReader(file);
      BufferedReader bufferedReader = new BufferedReader(fileReader);
      String line;
      while ((line = bufferedReader.readLine()) != null) {
             result.add(line);
      }
     } catch(IOException e) {
         e.printStackTrace();
     } finally {
        try {
         fileReader.close();
        } catch(IOException e) {
         e.printStackTrace();
        }
     }
     return result;
  }

  public boolean valida(String cat) {
     List<String> result = new ArrayList<String>();
     result=listaNombreCat();
     for(String st : result) {
        if(st.equals(cat)) {
          return true;
        }
     }
     return false;
  }
}
~~~

`DAOCategoryImplRedis.java`
~~~java
package productservlet;
import redis.clients.jedis.Jedis;
import java.util.ArrayList;
import java.util.List;

import javax.enterprise.inject.Alternative;

@Alternative
public class DAOCategoryImplRedis implements DAOCategory {
  public List<String> listaNombreCat() {
  //  List<String> result = new ArrayList<String>();
    Jedis conn = new Jedis("localhost");
    conn.select(4);
    return conn.lrange("listaCategory",0,-1);
  }
 
  public boolean valida(String cat) {
     List<String> result = new ArrayList<String>();
     result=listaNombreCat();
     for(String st : result) {
        if(st.equals(cat)) {
          return true;
        }
     }
     return false;
  }
}
~~~

Es decir al crear la interface **DAOCategory** podemos hacer uso de los métodos `public List<String> listaNombreCat();` `public boolean valida(String cat);`, de cualquiera de las dos anteriores implementaciones, que no hará falta recompilar las demás clases de este proyecto.


Otro uso de inyección de dependencia lo podemos encontrar a la hora de usar frameworks como Spring.

En el ejemplo visto en clase de la banda:

`App.java`
~~~java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {
	public static void main(String[] args) throws PerformanceException {
		ApplicationContext context = new ClassPathXmlApplicationContext(
				"SpringBeans.xml");

		Performer performer = (Performer) context.getBean("kenny");
		performer.perform();
	}
}
~~~
`SpringBeans.xml`
~~~xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">

	<bean id="kenny" class="OneManBandMap">
	  <property name="instruments">
	    <map>
	      <entry key="SAXO" value-ref="sax"/>
	      <entry key="PIAN" value-ref="pia"/>
	      <entry key="CORN" value-ref="cor"/>
	    </map>	
	  </property>
	</bean>
	<bean id="sax" class="Saxophone"/>
	<bean id="pia" class="Piano"/>
	<bean id="cor" class="Corneta"/>
</beans>
~~~

En **App.java** va a buscar en el bean las clases que debe usar, aquí encontramos la inyección de dependencia, ya que podríamos borrar `<entry key="CORN" value-ref="cor"/>` y no haría falta recompilar el código o en el caso de añadir otro instrumento p.e.`<bean id="tro" class="Trompeta"/>` `<entry key="TROM" value-ref="tro"/>`.

En resumen, con la inyección de dependencia nuestro código es más modular que esto a su vez nos facilita y ahorra tiempo en hacer test unitarios. También escribimos código más rápido al quitar la necesidad de tener que instanciar objetos.

Por otro lado, la inyección de dependencia puede ser tediosa, por la abstracción del código, al tener que crear una interfaz y sus implementaciones. Sin olvidar que tenemos que configurar los inyectores.




