# Studio_Autenticazione_multiruolo
## Studio di un sistema di autenticazione multiruolo


Il sistema prevede la gestione delle rotte di un applicativo a seconda del ruolo

- Un utente ha associati zero o più profili
- Ogni profilo ha associati zero o più settori
- Ogni settore di ogni profilo ha associati zero o più ruoli
- Ogni ruolo ha associate zero o più rotte



## Ogni utente alla creazione avrà assocato un albero di dati come il seguente


```JSON
{
    "user": {
        "id": 8,
        "username": "Tizio",
        "name": "Tizio",
        "cognome": "Rossi",
        "CF": "MRSFFT98G88I198F",
        "password": "passwordABC",
        "email": "trossi@esempio.it",
        "creator": "Admin",
        "id_creator": 1,
        "creation_date": "10/10/2010",
        "Active": false,
        "profile": null,
    }
}
```

che potrà essere popolato fino a diventare qualcosa del genere:

```JSON
{
    "user": {
        "id": 3,
        "username": "MarioRossi",
        "name": "Mario",
        "cognome": "Rossi",
        "CF": "MRSFGT98G33I198F",
        "password": "passwordABC",
        "email": "mrossi@esempio.it",
        "creator": "Admin",
        "id_creator": 1,
        "creation_date": "10/10/2010",
        "Active": true,
        "profile": {
            "id": 198,
            "name": "profilo di Mario Rossi",
            "description": "Una descrizione",
            "active": true,
            "defaultSector": 2,
            "beholder": [0, 1, 2, 3],
            "sectors": [
                {
                    "id": 1,
                    "name": "Sector A",
                    "description": "a sector description",
                    "reserved": true,
                    "roles": [
                        {
                            "id": 1,
                            "name": "Role A",
                            "routes": [
                                {
                                    "id": 1,
                                    "name": "Route A",
                                    "active": true
                                },
                                {
                                    "id": 3,
                                    "name": "Route B",
                                    "active": true
                                }
                            ]
                        },
                        {
                            "id": 2,
                            "name": "Role B",
                            "routes": [
                                {
                                    "id": 3,
                                    "name": "Route B",
                                    "active": true
                                },
                                {
                                    "id": 9,
                                    "name": "Route D",
                                    "active": false
                                }
                            ]
                        }
                    ]
                },
                {
                    "id": 2,
                    "name": "Sector B",
                    "description": "a sector description",
                    "reserved": true,
                    "roles": [
                        {
                            "id": 3,
                            "name": "Role C",
                            "routes": [
                                {
                                    "id": 2,
                                    "name": "Route E",
                                    "active": true
                                },
                                {
                                    "id": 18,
                                    "name": "Route F",
                                    "active": true
                                }
                            ]
                        },
                        {
                            "id": 2,
                            "name": "Role B",
                            "routes": [
                                {
                                    "id": 3,
                                    "name": "Route B",
                                    "active": true
                                },
                                {
                                    "id": 9,
                                    "name": "Route D",
                                    "active": false
                                }
                            ]
                        }
                    ]
                }
            ]
        }
    }
}
```


## Prima analisi - soluzione non completa e banale:

```python
# Tabella degli utenti
class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, autoincrement=True, index=True)
    username = Column(String(50), unique=True, index=True)
    name = Column(String(50))
    cognome = Column(String(50))
    CF = Column(String(16), unique=True, index=True)
    password = Column(String(100))
    email = Column(String(100), unique=True, index=True)
    superuser = Column(Boolean, unique=False, default=False)
    creator = Column(String(50))
    id_creator = Column(Integer, ForeignKey("users.id"), nullable=True)
    creation_date = Column(String(12))
    active = Column(Boolean, unique=False, default=False)

# Tabella dei profili
class Profile(Base):
    __tablename__ = "profiles"
    id = Column(Integer, primary_key=True, autoincrement=True, index=True)
    name = Column(String(50))
    description = Column(String(200))
    active = Column(Boolean, default=True)
    default_sector_id = Column(Integer, ForeignKey("sectors.id"), nullable=True)
    only_observer = Column(Boolean, unique=False, default=True)
    user_id = Column(Integer, ForeignKey("users.id"), nullable=True)

# Tabella dei settori
class Sector(Base):
    __tablename__ = "sectors"
    id = Column(Integer, primary_key=True, autoincrement=True, index=True)
    name = Column(String(50))
    description = Column(String(200))
    reserved = Column(Boolean, unique=False, default=True)
    profile_id = Column(Integer, ForeignKey("profiles.id"), nullable=True)

# Tabella dei ruoli
class Role(Base):
    __tablename__ = "roles"
    id = Column(Integer, primary_key=True, autoincrement=True, index=True)
    name = Column(String(60))
    sector_id = Column(Integer, ForeignKey("sectors.id"), nullable=True)

# Tabella delle rotte
class Route(Base):
    __tablename__ = "routes"
    id = Column(Integer, primary_key=True, autoincrement=True, index=True)
    name = Column(String(20))
    route = Column(String(20))
    description = Column(String(50))
    active = Column(Boolean, unique=False, default=True)
    role_id = Column(Integer, ForeignKey("roles.id"), nullable=True)
```


## Seconda analisi: qui si crea la necessità di fare un overlap nelle relazioni

```python
# Tabella degli utenti
class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    username = Column(String(50), unique=True, index=True)
    name = Column(String(50))
    cognome = Column(String(50))
    CF = Column(String(16), unique=True, index=True)
    password = Column(String(100))
    email = Column(String(100), unique=True, index=True)
    superuser = Column(Boolean, unique=False, default=False)
    creator = Column(String(50))
    id_creator = Column(Integer, ForeignKey("users.id"))
    creation_date = Column(String(10))
    active = Column(Boolean, unique=False, default=True)

    # Relazione con la tabella dei profili
    profile = relationship("Profile", uselist=False, back_populates="user")
    # Relazione con la tabella dei settori
    sectors = relationship("Sector", secondary="profiles", back_populates="users", join_depth=2)

    def as_dict(self):
        return {c.name: getattr(self, c.name) for c in self.__table__.columns}

# Tabella dei profili
class Profile(Base):
    __tablename__ = "profiles"
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(50))
    description = Column(String(200))
    active = Column(Boolean, default=True) #indica se l'utente è attivo
    default_sector_id = Column(Integer, ForeignKey("sectors.id")) # settore di default
    only_observer = Column(Boolean, unique=False, default=True) # se è un osservatore imposta la visione (settore di default)
    user_id = Column(Integer, ForeignKey("users.id"))
    user = relationship("User", back_populates="profile")
    sectors = relationship("Sector", secondary="profile_sector_associations", back_populates="profiles")

    def as_dict(self):
        return {c.name: getattr(self, c.name) for c in self.__table__.columns}

# Tabella di associazione tra profili e settori
profile_sector_association = Table('profile_sector_associations', Base.metadata,
    Column('profile_id', Integer, ForeignKey('profiles.id')),
    Column('sector_id', Integer, ForeignKey('sectors.id'))
)


# Tabella dei settori
class Sector(Base):
    __tablename__ = "sectors"
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(50))
    description = Column(String(200))
    reserved = Column(Boolean, unique=False, default=True) # impedisce di leggere le rotte a un utente non assegnato a quel settore
    profiles = relationship("Profile", secondary="profile_sector_associations", back_populates="sectors")
    users = relationship("User", secondary="profiles", back_populates="sectors", join_depth=2)
    roles = relationship("Role", back_populates="sector")
    def as_dict(self):
        return {c.name: getattr(self, c.name) for c in self.__table__.columns}

# Tabella dei ruoli
class Role(Base):
    __tablename__ = "roles"
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(50))
    #sector_id = Column(Integer, ForeignKey("sectors.id")) rimosso
    sector = relationship("Sector", back_populates="roles")
    user_roles = relationship("UserRole", back_populates="role")
    route_roles = relationship("RouteRoleAssociation", back_populates="role")
    def as_dict(self):
        return {c.name: getattr(self, c.name) for c in self.__table__.columns}

# Tabella di associazione utenti-ruoli
class UserRole(Base):
    __tablename__ = "user_roles"
    user_id = Column(Integer, ForeignKey("users.id"), primary_key=True)
    role_id = Column(Integer, ForeignKey("roles.id"), primary_key=True)
    sector_id = Column(Integer, ForeignKey("sectors.id"))
    user = relationship("User", back_populates="user_roles") #controllare
    role = relationship("Role", back_populates="user_roles")
    sector = relationship("Sector")

    uq_user_role_sector = UniqueConstraint('user_id', 'sector_id', 'role_id', name='uq_user_role_sector')

    def as_dict(self):
        return {c.name: getattr(self, c.name) for c in self.__table__.columns}


# Tabella delle rotte
class Route(Base):
    __tablename__ = "routes"
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(50))
    active = Column(Boolean, unique=False, default=True)
    route_roles = relationship("RouteRoleAssociation", back_populates="route")
    def as_dict(self):
        return {c.name: getattr(self, c.name) for c in self.__table__.columns}

# Tabella di associazione ruoli-rotte
class RouteRoleAssociation(Base):
    __tablename__ = "route_role_associations"
    role_id = Column(Integer, ForeignKey("roles.id"), primary_key=True)
    route_id = Column(Integer, ForeignKey("routes.id"), primary_key=True)
    role = relationship("Role", back_populates="route_roles")
    route = relationship("Route", back_populates="route_roles")
    def as_dict(self):
        return {c.name: getattr(self, c.name) for c in self.__table__.columns}
```

## Terza analisi del problema

```python

# Definizione della tabella di associazione tra utenti e ruoli
user_roles = Table(
    'user_roles', 
    Base.metadata,
    Column('user_id', Integer, ForeignKey('users.id')),
    Column('role_id', Integer, ForeignKey('roles.id'))
)

# Definizione della tabella di associazione tra ruoli e rotte
role_routes = Table(
    'role_routes',
    Base.metadata,
    Column('role_id', Integer, ForeignKey('roles.id')),
    Column('route_id', Integer, ForeignKey('routes.id'))
)

# Definizione della tabella di associazione tra utenti e rotte
user_routes = Table(
    'user_routes',
    Base.metadata,
    Column('user_id', Integer, ForeignKey('users.id')),
    Column('route_id', Integer, ForeignKey('routes.id'))
)

class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, autoincrement=True, primary_key=True, index=True)
    name = Column(String, index=True)
    email = Column(String, unique=True, index=True)
    loginUsername = Column(String(50), unique=True, index=True)
    password = Column(String, nullable=False)
    roles = relationship('Role', secondary=user_roles, back_populates='users')
    routes = relationship('Route', secondary=user_routes, back_populates='users')


class Role(Base):
    __tablename__ = 'roles'
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True, unique=True)
    users = relationship('User', secondary=user_roles, back_populates='roles')
    routes = relationship('Route', secondary=role_routes, back_populates='roles')

class Route(Base):
    __tablename__ = 'routes'
    id = Column(Integer, primary_key=True, autoincrement=True, index=True) #id univoco della rotta
    name = Column(String, index=True, unique=True) #Nome della rotta
    path = Column(String, unique=True) #percorso relativo della rotta ad esempio /rotta1/sottoRotta1
    note = Column(String) #note opzionali relative alla rotta
    active = Column(Boolean=True) #indica se la rotta è attiva
    roles = relationship('Role', secondary=role_routes, back_populates='routes')
    users = relationship('User', secondary=user_routes, back_populates='routes')
 ```
