!SLIDE 
# TDD: Test-driven development #

!SLIDE subsection transition=scrollUp
# Unit tests #

!SLIDE bullets incremental transition=scrollUp
# Pruebas unitarias #

* Automatizar las pruebas
* Probar por "pedacitos"
* Definir "estándares" o "especificaciones"
* Sin dependencias

!SLIDE bullets incremental transition=scrollUp
# Escenarios #

* Happy path
* Parámetros de entrada incorrectos
* Errores en servicios / dependencias
* Manejo de excepciones
* Etc.

!SLIDE execute small transition=scrollUp
# Ejemplo #

    @@@ java
    @Transactional
    @Override
    public Mesorregion update(Mesorregion entity) {
        if (entity != null) {
            entity.setFechaModificacion(new Date());
            return super.update(entity);
        } else {
            throw new IllegalArgumentException(
                    DaoErrors.NULL_PARAM);
        }
    }

!SLIDE execute transition=scrollUp
# Mocks #

    @@@ java
    Mockery context = new JUnit4Mockery();
    EntityManager em =
        context.mock(EntityManager.class);
    Query query = context.mock(Query.class);

!SLIDE execute small transition=scrollUp
# Happy path #

    @@@ java
    @Test
    public void update() {
        final Long id = new Long(1);
        final Mesorregion mesorregion = 
                new Mesorregion(id, "Centro");

        context.checking(new Expectations() {{
            oneOf (em).merge(mesorregion);
                will(returnValue(mesorregion));
        }});

        Mesorregion result = mesorregionDao.update(mesorregion);
        assertNotNull(result);
        assertNotNull(result.getFechaModificacion());
    }

!SLIDE execute small transition=scrollUp
# Parámetros / Excepciones #

    @@@ java
    @Test
    public void updateNull() {
        try {
            mesorregionDao.update(null);
            fail("Debería lanzar una excepción");
        } catch (IllegalArgumentException e) {
            assertEquals(DaoErrors.NULL_PARAM, e.getMessage());
        }
    }

!SLIDE execute small transition=scrollUp
# Lógica "interna" #

    @@@ java
    context.checking(new Expectations() {{
        oneOf (em).createNamedQuery("mesorregion.active.findAll");
            will(returnValue(query));
        oneOf (query).setMaxResults(AbstractJpaDao.DEFAULT_PAGE_SIZE);
            will(returnValue(query));
        oneOf (query).setFirstResult(0);
            will(returnValue(query));
        oneOf (query).getResultList();
            will(returnValue(results));
        oneOf (em).createQuery(
            with(allOf(
                containsString("COUNT"),
                containsString("Mesorregion"))));
            will(returnValue(query));
        oneOf (query).getSingleResult();
        will(returnValue(count));
    }});

    SearchResult<Mesorregion> resultados =
        mesorregionDao.findAll(new PaginatedSearchArg(-5, -5));

!SLIDE execute transition=scrollUp
# Cómo ejecutar las pruebas #

## Deben terminar en _Test_ ##

    @@@ shell
    mvn test

    mvn test -pl dao -am

