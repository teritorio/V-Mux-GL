# V-Mux-GL
Vector Tiles and Style GL Multiplexer.

Style
- Patch style on the fly. Add, remove or replace style layer. Changes come from patch file or from patch derived from other styles. Report style enhancement from one style to an other.

Vector Tiles
- Add data layers from other sources.
- Replace selected data in data layer from one source by an other source.

Affect Style and Vector Tiles user by user using API Keys.


## Build
```
docker compose --env-file .tools.env build
```

## Configuration

`config.yaml`

```yaml
sources:
    foo:
        key: fi787or6ej8famrejfffp

        sources:
            full:
                tilejson_url: https://vecto-dev.teritorio.xyz/data/teritorio-dev.json
                tile_url: http://localhost:3000
            partial:
                mbtiles: restaurent-20200819.mbtiles

        merge_layers:
            poi_tourism:
                fields: [superclass, class, subclass]
            features_tourism:

        output:
            min_zoom: 14

        styles:
            teritorio-tourism-0.9:
                url: https://vecto.teritorio.xyz/styles/teritorio-tourism-0.9/style.json
                merged_source: openmaptiles
                # Patch the style
                layers_patch:
                    # Patch from local diff file
                -   diff:
                        patch: features.layers-patch.json
                    amend:
                        add:
                        -   id: features-fill
                            insert_before_id: building
                    # Diff3, patch generated by diff on other styles
                -   diff:
                        from: https://vecto.teritorio.xyz/styles/teritorio-basic-dev/style.json
                        to: https://vecto.teritorio.xyz/styles/teritorio-tourism-dev/style.json
                # Add layers
                layers:
                -   insert_before_id: building
                    layer: &features-outline
                        id: features-outline
                        type: line
                        source: openmaptiles
                        source-layer: features
                        paint:
                            line-color: #ff0000

        cache:
            prefetch: false # Optional cache prefetch disable, default: true

fetch_http_headers:
    Secret: foo

server:
    config_path: ./config/
    public_base_path:
    public_tile_url_prefixes: []
```

## Initialize data
Setup configuration in `data` and fetch data:
```
docker compose --env-file .tools.env run --rm fetcher
```

## Run
```
docker compose up -d
```

Fill the tiles cache in nginx:
```
docker compose --env-file .tools.env run --rm expire
```

## Data update

Get and switch to new data:
```
docker compose --env-file .tools.env run --rm fetcher
docker compose restart
```

Partial fetch
```
docker compose --env-file .tools.env run --rm fetcher bash -c 'ruby ./fetcher.rb foo'
```

Update the tiles cache in nginx:
```
docker compose --env-file .tools.env run --rm expire
```

Partial expire
```
docker compose --env-file .tools.env run --rm expire sh -c 'ruby ./get_tiles.rb foo'
```

## Serve

Under reverse proxy HTTP header `Host` should contains the original value.
Header `Forwarded` should also properly set. See https://www.nginx.com/resources/wiki/start/topics/examples/forwarded/


# Dev

## Test

```
docker compose --env-file .dev.env run --rm merge_proxy bash
```
