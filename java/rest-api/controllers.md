```java
package com.ids.contact_ms.web;

import com.ids.contact_ms.entity.ActivitySector;
import com.ids.contact_ms.service.ActivitySectorService;
import lombok.Getter;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Pageable;
import org.springframework.data.web.PagedModel;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RequiredArgsConstructor
@RestController
@RequestMapping("/api/v1/activity-sectors")
public class ActivitySectorController {

    @Getter
    private final ActivitySectorService service;

    @GetMapping
    public PagedModel<ActivitySector> find(@RequestParam(defaultValue = "false") boolean archived, Pageable pageable) {
        return service.find(archived, pageable);
    }

    @GetMapping("/{id}")
    public ActivitySector findOne(@PathVariable Long id) {
        return service.findOne(id);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ActivitySector create(@RequestBody ActivitySector object) {
        return service.create(object);
    }

    @PutMapping("/{id}")
    public void update(@PathVariable Long id, @RequestBody ActivitySector object) {
        service.update(id, object);
    }

    @PatchMapping("/{id}/archive")
    public void archive(@PathVariable Long id) {
        service.archive(id);
    }

    @PatchMapping("/{id}/restore")
    public void restore(@PathVariable Long id) {
        service.restore(id);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable Long id) {
        service.delete(id);
    }
}

```
