2. Create a OSGI configuration service and include one string variable. This variable should have the value for author environment as " This servlet is executed from author environment" and for publisher the value of the variable should be " This servlet is executed from publisher environment". Use this configuration service in the above servlet and read the value fron configuration service and print this value as servlet output. When we execute the servlet on author and publisher it should print output as configured.

->  step 1: create a configuration interface

package com.ttn.bootcamp.core.configuration;

import org.osgi.service.metatype.annotations.AttributeDefinition;
import org.osgi.service.metatype.annotations.ObjectClassDefinition;

@ObjectClassDefinition(name = "Bootcamp OSGi Configuration Service")
public @interface BootcampOSGiConfiguration {

        @AttributeDefinition(name = "Environment Message", required = true,

        description = "Message to specify the environment.")

        String env_message() default "";
}

    Step 2: Create a service interface

    package com.ttn.bootcamp.core.services;

public interface MySimpleService {
    String fetchEnvMessage();
}

    Step 3: Create a service implementation class

    package com.ttn.bootcamp.core.services.impl;


import com.ttn.bootcamp.core.configuration.BootcampOSGiConfiguration;
import com.ttn.bootcamp.core.services.MySimpleService;
import org.osgi.service.component.annotations.Activate;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Deactivate;
import org.osgi.service.component.annotations.Modified;
import org.osgi.service.metatype.annotations.Designate;

@Component(service = MySimpleService.class, immediate = true)
@Designate(ocd = BootcampOSGiConfiguration.class)
public class MySimpleServiceImpl implements MySimpleService {

    private String envMessage;

    @Activate
    @Modified
    public void activate(BootcampOSGiConfiguration config) {
        envMessage = config.env_message();
    }

    @Deactivate
    public void deactivate(BootcampOSGiConfiguration config) {
        envMessage = "";
    }

    @Override
    public String fetchEnvMessage() {
        return envMessage;
    }
}

    Step 4: Create a servlet class that can use above created service

    package com.ttn.bootcamp.core.servlets;

import com.ttn.bootcamp.core.services.MySimpleService;
import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.api.servlets.HttpConstants;
import org.apache.sling.api.servlets.SlingAllMethodsServlet;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;

import javax.servlet.Servlet;
import javax.servlet.ServletException;
import java.io.IOException;

@Component(service = { Servlet.class }, property = {
        "sling.servlet.methods="+HttpConstants.METHOD_GET,
        "sling.servlet.paths=/bin/bootcamp/configtest",
        "sling.servlet.extension=html",

})
public class ConfigTestServlet extends SlingAllMethodsServlet {

    @Reference
    MySimpleService mySimpleService;

    @Override
    protected void doGet(SlingHttpServletRequest request, SlingHttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html");
        response.getWriter().write("<h1>This is my first servlet.</h1>\n Environment message"+ mySimpleService.fetchEnvMessage());
    }
}

    Step 5 create a config json file under ui.config/**/config.author path with name <package> + <service name which is using configuration class> + ~ + <app name> + cfg.json e.g com.ttn.bootcamp.core.services.impl.MySimpleServiceImpl~ttnBootcamp.cfg.json in out case with the below json

    {
  "env.message": "This servlet is executed from author environment"
}

    Step 6 create a config json file under ui.config/**/config.publish path with name <package> + <service name which is using configuration class> + ~ + <app name> + cfg.json e.g com.ttn.bootcamp.core.services.impl.MySimpleServiceImpl~ttnBootcamp.cfg.json in out case with the below json

    {
  "env.message": "This servlet is executed from publisher environment"
}


