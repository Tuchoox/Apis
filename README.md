"ConnectionStrings": {
  "defaultConnection": "Data Source=DESKTOP-A7V2SVV\\SQLEXPRESS;Initial Catalog=db_envios_UBUNTU;Integrated Security=True;Encrypt=False"
},

builder.Services.AddDbContext<db_enviosContext>(config => config.UseSqlServer(builder.Configuration.GetConnectionString("defaultConnection")));
builder.Services.AddScoped<IEnvioRepository, EnvioRepository>();


db_enviosContext _context;
public EnvioRepository(db_enviosContext context)
{
    _context = context;
}

public bool Create(TEnvio envio)
{
    _context.TEnvios.Add(envio);
    return _context.SaveChanges() == 1;
}

public bool Delete(int id)
{
    var eliminado = GetById(id);
    _context.TEnvios.Remove(eliminado);
    return _context.SaveChanges() == 1;
}

public bool DeleteEstado(int id)
{
    var envio = GetById(id);
    if (envio.Estado != "cancelado")
    {
        envio.Estado = "cancelado";
        _context.TEnvios.Update(envio);
    }
    return _context.SaveChanges() == 1;
}

public List<TEnvio> GetAll()
{
    return _context.TEnvios.ToList();
}

public TEnvio? GetById(int id)
{
    return _context.TEnvios.Find(id);
}

public List<TEnvio> GetByUBUNTU(DateTime fec_desde, DateTime fec_hasta)
{
    return _context.TEnvios.Where(x => x.FechaEnvio >= fec_desde && x.FechaEnvio <= fec_hasta && x.Estado != "cancelado").ToList();
}

public bool Update(TEnvio envio)
{
    _context.TEnvios.Update(envio);
    return _context.SaveChanges() == 1;
}




IEnvioRepository _repository;
public EnviosController(IEnvioRepository repository)
{
    _repository = repository;
}

// GET: api/<EnviosController>
[HttpGet]
public IActionResult Get([FromQuery] DateTime fec_desde, DateTime fec_hasta)
{
    try
    {
        return Ok(_repository.GetByUBUNTU(fec_desde,fec_hasta));
    }
    catch (Exception ex)
    {
        return BadRequest($"mal{ex}");
    }
}

// POST api/<EnviosController>
[HttpPost]
public IActionResult Post([FromBody] TEnvio envio)
{
    try
    {
        if (IsValid(envio))
        {
            _repository.Create(envio);
            return Ok("nice");
        }
        else
        {
            return StatusCode(400,"malll");
        }
    }
    catch (Exception ex)
    {
        return BadRequest($"nt{ex}");
    }
}

private bool IsValid(TEnvio envio)
{
    return envio.FechaEnvio > DateTime.Today
        && !envio.Direccion.IsNullOrEmpty()
        && !envio.Estado.IsNullOrEmpty()
        && !envio.DniCliente.IsNullOrEmpty()
        && envio.IdEmpresa > 0;
}

// DELETE api/<EnviosController>/5
[HttpDelete("CambioEstado/{id}")]
public IActionResult DeleteEstado(int id)
{
    try
    {
        if (_repository?.DeleteEstado(id) == true)
        {
            return Ok("nice");
        }
        else
        {
            return NotFound("omg nooo existe o ya esta cancelado");
        }

    }
    catch (Exception ex)
    {
        return BadRequest($"nttt mann{ex}");
    }
}

[HttpDelete("Eliminar/{id}")]
public IActionResult Delete(int id)
{
    try
    {
        if (_repository?.Delete(id) == true)
        {
            return Ok("nice");
        }
        else
        {
            return NotFound("omg nooo");
        }

    }
    catch (Exception ex)
    {
        return BadRequest($"nttt mann{ex}");
    }
}

[HttpPut("{id}")]
public IActionResult Put(int id,[FromBody]TEnvio envio)
{
    try
    {
        var oEnvio = _repository.GetById(id);
        if (oEnvio != null && IsValid(envio))
        {
            oEnvio.FechaEnvio = envio.FechaEnvio;
            oEnvio.Direccion = envio.Direccion;
            oEnvio.Estado = envio.Estado;
            oEnvio.DniCliente = envio.DniCliente;
            oEnvio.IdEmpresa = envio.IdEmpresa;
            _repository.Update(oEnvio);
            return Ok("excelente manoo");
        }
        else
        {
            return BadRequest("muito malo mano");
        }
    }
    catch(Exception ex)
    {
        return BadRequest($"nt mannnn{ex}");
    }
}

//[HttpPut]
//public IActionResult Put([FromBody] TEnvio value)
//{
//    try
//    {
//        if (value.Codigo != null && IsValid(value))
//        {
//            _repository.Update(value);
//            return Ok("Envio Actualizado Con Exito!!");
//        }
//        else
//        {
//            return BadRequest("Datos Incompletos o incorrectos");
//        }
//    }
//    catch (Exception)
//    {
//        return StatusCode(500, "Error Interno");
//    }
//}
