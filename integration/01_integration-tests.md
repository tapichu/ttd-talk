!SLIDE subsection transition=scrollUp
# Integration Tests #

!SLIDE bullets incremental transition=scrollUp
# Pruebas de integración #

* Probar con dependencias:
* Contexto de spring
* Base de datos
* Transacciones (JTA)
* Etc.

!SLIDE bullets incremental transition=scrollUp
# Escenarios #

* Validación de entradas y salidas
* Probar físicamente en la BD
* Revisar el resultado de las transacciones
* Pruebas de rendimiento
* Etc.

!SLIDE execute smaller transition=scrollUp
# Ejemplo #

    @@@ java
    @Transactional
    @Override
    public Mesorregion create(Mesorregion mesorregion) {
        if (mesorregion != null) {
            mesorregion.setFechaCreacion(new Date());
            mesorregion.setActivo(true);

            Map<String, Object> params = new HashMap<String, Object>();
            params.put("nombre", mesorregion.getNombre());
            List<Mesorregion> mesorregiones =
                queryNamed("mesorregion.inactive.findByName", params);
            if (mesorregiones != null && !mesorregiones.isEmpty()) {
                mesorregion.setId(mesorregiones.get(0).getId());
                mesorregion.setFechaModificacion(null);
                return super.update(mesorregion);
            }
            return super.create(mesorregion);
        } else {
            throw new IllegalArgumentException(DaoErrors.NULL_PARAM);
        }
    }

!SLIDE execute transition=scrollUp
# Preparar la BD #

    @@@ java
    @Override
    public String getXmlDataFile() {
        return "src/test/resources/" +
            "META-INF/data/mesorregion.xml";
    }

!SLIDE execute small transition=scrollUp
# Validar Salidas #

    @@@ java
    @Test
    public void create() {
        Mesorregion mesorregion = new Mesorregion("Este");
        mesorregion = mesorregionDao.create(mesorregion);

        assertNotNull(mesorregion.getId());
        assertEquals(1l, mesorregion.getId().longValue());
        assertEquals("Este", mesorregion.getNombre());
        assertTrue(mesorregion.isActivo());
        // Debe tener fecha de creación
        assertNotNull(mesorregion.getFechaCreacion());
        assertNull(mesorregion.getFechaModificacion());

        assertEquals(4, mesorregionDao.findAll().size());
        // Evitar 'false positives'!!!
        entityManager.flush();
    }

!SLIDE execute small transition=scrollUp
# Validar errores / transacciones #

    @@@ java
    @Test
    @Rollback(true)
    @ExpectedException(value=DataAccessException.class)
    public void createNull() {
        mesorregionDao.create(null);
    }

!SLIDE execute small transition=scrollUp
# Probar rendimiento #

    @@@ java
    @Test
    @Repeat(1000)
    @Timed(200)
    public void create() {
        Mesorregion mesorregion = new Mesorregion("Este");
        mesorregion = mesorregionDao.create(mesorregion);

        assertNotNull(mesorregion.getId());
        assertEquals(1l, mesorregion.getId().longValue());
        assertEquals("Este", mesorregion.getNombre());
        assertTrue(mesorregion.isActivo());
        // Debe tener fecha de creación
        assertNotNull(mesorregion.getFechaCreacion());
        assertNull(mesorregion.getFechaModificacion());

        assertEquals(4, mesorregionDao.findAll().size());
        entityManager.flush();
    }

!SLIDE execute transition=scrollUp
# Cómo ejecutar las pruebas #

## Deben terminar en _IT_ ##

    @@@ shell
    mvn verify

    mvn verify -pl dao -am

    mvn verify -pl dao -am -Dit.test=*Meso*IT

    mvn verify -Dit.test=*Meso*IT,*Esta*IT

