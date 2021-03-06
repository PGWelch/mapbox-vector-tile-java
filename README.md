# MapBox Vector Tile - Java

## Overview

Provides:

 * protoc generated model for Mapbox Vector Tiles v2.1.
 * Provides MVT encoding through use of the Java Topology Suite (JTS).

See:

 * https://github.com/mapbox/vector-tile-spec/tree/master/2.1 for details on the MVT spec.
 * https://developers.google.com/protocol-buffers/ for details on protoc.
 * http://www.vividsolutions.com/jts/ for details on JTS.

### Dependency

#### Maven

```xml
<dependency>
    <groupId>com.wdtinc</groupId>
    <artifactId>mapbox-vector-tile</artifactId>
    <version>1.1.1</version>
</dependency>
```

#### Gradle

```
compile 'com.wdtinc:mapbox-vector-tile:1.1.1'
```

### Reading MVTs

Use MvtReader.loadMvt() to load MVT data from a path or input stream
into JTS geometry. The TagKeyValueMapConverter instance will convert
MVT feature tags to a Map with primitive values. The map will be
stored as a JTS geometry user data object within the Geometry.

```java
GeometryFactory geomFactory = new GeometryFactory();

List<Geometry> geoms = MvtReader.loadMvt(
        Paths.get("path/to/your.mvt"),
        geomFactory,
        new TagKeyValueMapConverter());


// Allow negative-area exterior rings with classifier
List<Geometry> geoms = MvtReader.loadMvt(
        Paths.get("path/to/your.mvt"),
        geomFactory,
        new TagKeyValueMapConverter(),
        MvtReader.RING_CLASSIFIER_V1);
```

### Building and Writing MVTs

#### 1) Create or Load JTS Geometry

Create or load any JTS Geometry that will be included in the MVT.

#### 2) Create Tiled JTS Geometry in MVT Coordinates

Create tiled JTS geometry with JtsAdapter#createTileGeom(). MVTs currently
do not support feature collections so any JTS geometry will be flattened
to a single level. A TileGeomResult will contain the intersection
geometry from clipping as well  as the actual MVT geometry that uses
extent coordinates. The intersection geometry can be used for hierarchical
processing.

```java
IGeometryFilter acceptAllGeomFilter = geometry -> true;
Envelope tileEnvelope = new Envelope(0d, 100d, 0d, 100d); // TODO: Your tile extent here
MvtLayerParams layerParams = new MvtLayerParams(); // Default extent

TileGeomResult tileGeom = JtsAdapter.createTileGeom(
        jtsGeom, // Your geometry
        tileEnvelope,
        geomFactory,
        layerParams,
        acceptAllGeomFilter);
```

#### 3) Create MVT Builder, Layers, and Features

Create the VectorTile.Tile.Builder responsible for the MVT protobuf
byte array:

```java
VectorTile.Tile.Builder tileBuilder = VectorTile.Tile.newBuilder();
```

Create an empty layer for the MVT:

```java
VectorTile.Tile.Layer.Builder layerBuilder = MvtLayerBuild.newLayerBuilder("myLayerName", layerParams);
```

MVT JTS Geometry can be converted to MVT features.

MvtLayerProps is a supporting class for building MVT layer
key/value dictionaries. A user data converter will take JTS Geometry
user data and convert it to MVT tags:

```java
MvtLayerProps layerProps = new MvtLayerProps();
IUserDataConverter userDataConverter = new UserDataKeyValueMapConverter();
List<VectorTile.Tile.Feature> features = JtsAdapter.toFeatures(tileGeom.mvtGeoms, layerProps, userDataConverter);
```

Use MvtLayerBuild#writeProps() to add the key/value dictionary to the
MVT layer.

```java
MvtLayerBuild.writeProps(layerBuilder, layerProps);
```

#### 4) Write MVT

```java
VectorTile.Tile mvt = tileBuilder.build();
try {
    Files.write(path, mvt.toByteArray());
} catch (IOException e) {
    logger.error(e.getMessage(), e);
}
```

## Examples

See tests.

## How to generate VectorTile class using vector_tile.proto

If vector_tile.proto is changed in the specification, VectorTile may need to be regenerated.

Command `protoc` version should be the same version as the POM.xml dependency.

protoc --java_out=src/main/java src/main/resources/vector_tile.proto

#### Extra .proto config

These options were added to the .proto file:

 * syntax = "proto2";
 * option java_package = "com.wdtinc.mapbox_vector_tile";
 * option java_outer_classname = "VectorTile";

## Issues

#### Reporting

Use the Github issue tracker.

#### Known Issues

 * Creating tile geometry with non-simple line strings that self-cross in many places will be 'noded' by JTS during an intersection operation. This results in ugly output.
 * Invalid or non-simple geometry may not work correctly with JTS operations when creating tile geometry.

## Contributing

See [CONTRIBUTING](CONTRIBUTING.md)
 
## License

http://www.apache.org/licenses/LICENSE-2.0.html