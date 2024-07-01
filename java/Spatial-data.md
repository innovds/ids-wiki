# Working with Spatial Data using Spring Web, Spring Data JPA, and Postgres

This guide provides a step-by-step approach to handling spatial data using the Spring Web framework, Spring Data JPA, and PostgreSQL with the **PostGIS** extension. The examples included demonstrate how to create, list, and manage spatial data, specifically focusing on the integration with Spring Boot and Hibernate Spatial.

---

## Dependencies

Add the following dependencies to your `pom.xml` to enable Hibernate Spatial and JTS (Java Topology Suite) for handling spatial data:

```xml
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-spatial</artifactId>
</dependency>
<dependency>
    <groupId>org.locationtech.jts</groupId>
    <artifactId>jts-core</artifactId>
    <version>1.19.0</version>
</dependency>
```

## Entity Setup

Define your entity to include a spatial column. Here is an example of a `ContactAddress` entity with a location field of type `Point`:

```java
package com.ids.contact_ms.entity;

import com.fasterxml.jackson.annotation.JsonIgnore;
import jakarta.persistence.*;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import lombok.experimental.SuperBuilder;
import org.locationtech.jts.geom.Point;

@Entity
@SuperBuilder(toBuilder = true)
@RequiredArgsConstructor
@Data
public class ContactAddress {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 60)
    private String defaultUsage;

    @Column(length = 120)
    private String streetName;

    @Column(length = 120)
    private String streetNumber;

    @Column(length = 120)
    private String addressLine2;

    @Column(length = 40)
    private String city;

    @Column(length = 40)
    private String state;

    @Column(length = 40)
    private String country;

    @Column(length = 20)
    private String postalCode;

    @Column(columnDefinition = "geography(Point, 4326)")
    private Point location;

    @Enumerated(EnumType.STRING)
    private Status status;

    @JsonIgnore
    @ManyToOne(fetch = FetchType.LAZY)
    private Contact contact;
}
```

## Custom JSON Serialization/Deserialization

Create a custom JSON serializer and deserializer for the `Point` type to handle the conversion between JSON and the JTS `Point` object.

```java
package com.ids.contact_ms.commons.jackson.databind;

import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.*;
import org.locationtech.jts.geom.Coordinate;
import org.locationtech.jts.geom.GeometryFactory;
import org.locationtech.jts.geom.Point;

import java.io.IOException;

public class PointJsonConverter {

    public static final String LATITUDE = "latitude";
    public static final String LONGITUDE = "longitude";

    public static class PointSerializer extends JsonSerializer<Point> {
        @Override
        public void serialize(Point point, JsonGenerator jsonGenerator, SerializerProvider serializers) throws IOException {
            jsonGenerator.writeStartObject();
            jsonGenerator.writeNumberField(LATITUDE, point.getY());
            jsonGenerator.writeNumberField(LONGITUDE, point.getX());
            jsonGenerator.writeEndObject();
        }
    }

    public static class PointDeserializer extends JsonDeserializer<Point> {
        GeometryFactory factory = new GeometryFactory();
        @Override
        public Point deserialize(JsonParser jsonParser, DeserializationContext context) throws IOException, JsonProcessingException {
            JsonNode node = jsonParser.getCodec().readTree(jsonParser);
            double latitude = node.get(LATITUDE).asDouble();
            double longitude = node.get(LONGITUDE).asDouble();
            return factory.createPoint(new Coordinate(longitude, latitude));
        }
    }
}
```

## Configuration

Configure Jackson to use the custom serializer and deserializer:

```java
package com.ids.contact_ms.config;

import com.fasterxml.jackson.databind.Module;
import com.fasterxml.jackson.databind.module.SimpleModule;
import com.ids.contact_ms.commons.jackson.databind.PointJsonConverter;
import org.locationtech.jts.geom.Point;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.json.Jackson2ObjectMapperBuilder;

@Configuration
public class JacksonConfig {

    @Bean
    public Jackson2ObjectMapperBuilder jacksonBuilder() {
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder();
        builder.modulesToInstall(PointModule());
        return builder;
    }

    @Bean
    public Module PointModule() {
        SimpleModule module = new SimpleModule();
        module.addSerializer(Point.class, new PointJsonConverter.PointSerializer());
        module.addDeserializer(Point.class, new PointJsonConverter.PointDeserializer());
        return module;
    }
}
```

## Controller Example

Here's an example of a controller that handles CRUD operations for the `ContactAddress` entity:

```java
package com.ids.contact_ms.controller;

import com.ids.contact_ms.entity.ContactAddress;
import com.ids.contact_ms.repository.ContactAddressRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/v1/contacts")
public class ContactAddressController {

    @Autowired
    private ContactAddressRepository repository;

    @PostMapping
    public ContactAddress createContact(@RequestBody ContactAddress contactAddress) {
        return repository.save(contactAddress);
    }

    @GetMapping
    public List<ContactAddress> listContacts() {
        return repository.findAll();
    }
}
```

## Example Requests

### Creating a Contact

```bash
curl -X 'POST' \
  'http://localhost:8080/api/v1/contacts' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
        "defaultUsage": "Headquarters2",
        "streetName": "street",
        "streetNumber": "10",
        "addressLine2": "Suite 500",
        "city": "New York",
        "state": "NY",
        "country": "USA",
        "postalCode": "10001",
        "location": {
          "latitude": -70.006,
          "longitude": 43.7128
        }
      }'
```

### Listing Contacts

Example response:

```json
[
  {
    "id": 7,
    "archived": false,
    "createdDate": null,
    "createdBy": null,
    "defaultUsage": "Branch Office",
    "streetName": null,
    "streetNumber": null,
    "addressLine2": "",
    "city": "San Francisco",
    "state": "CA",
    "country": "USA",
    "postalCode": "94108",
    "location": {
      "latitude": -57.580600000000004,
      "longitude": 37.7749
    },
    "status": "ACTIVE",
    "new": false
  },
  {
    "id": 8,
    "archived": false,
    "createdDate": null,
    "createdBy": null,
    "defaultUsage": "Headquarters2",
    "streetName": "street",
    "streetNumber": "10",
    "addressLine2": "Suite 500",
    "city": "New York",
    "state": "NY",
    "country": "USA",
    "postalCode": "10001",
    "location": {
      "latitude": -70.006,
      "longitude": 43.7128
    },
    "status": null,
    "new": false
  }
]
```
