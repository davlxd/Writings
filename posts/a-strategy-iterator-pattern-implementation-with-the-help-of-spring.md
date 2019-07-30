# A Strategy & Iterator Pattern Implementation with the Help of Spring
2019-07-29

Recently I came across where a CSV file needs to be translated and converted into a Stream of Object, and the rules for the translation vary depending on the values in the CSV files. There are around 20 rules very well depicted in a large documentation. And I needed to design a structure that allows ~~me to get everyone involved in this tedious job~~ us to divide and conquer the work.

## Interface Oriented Programming

In the first step, I declared an interface to host all rule implementations:

```Java
@FunctionalInterface
public interface AssetTypeTrait {
  Optional<Asset> typePredicate(CsvRow csvRow);
}
```

Then for the dear teammate who's willing to lend a hand, all (s)he need to do is picking an unimplemented rule from the documentation, then create a corresponding implementing class from it, nice and easy.

```Java
@Service
public class GoldAssetImpl implements AssetTypeTrait {
  @Override
  public Optional<Asset> typePredicate(CsvRow csvRow) {
    if (StringUtils.substringBefore( ... {
      RegExUtils.removeFirst( ...
        // ....
      return Optional.of(asset);
    }
    return Optional.empty();
  }
}
```


## Aggregate classes implementing the same interface

Now we have a stack of implementing classes sitting in the same package, how do we aggregate them together, of course, in runtime. I was thinking to scan CLASSPATH or find them all out form SpringContext, however it turns I just need to inject against a `List<AssetTypeTrait>`, nice and easy.

```Java
@Service
public class ParsingService {
  @Autowired
  private List<AssetTypeTrait> assetTypeTraitList;

  //....
}
```

With the `List` of rules in place, the core parsing logic can be accomplished by some simple Java 8 stream programming, nice an easy.

```Java
@Service
public class ParsingService {
  public Optional<Asset> findAsset(CsvRow csvRow) {
    List<Asset> potentialAssetList = assetTypeTraitList.stream()
      .map(assetTypeTrait -> assetTypeTrait.typePredicate(csvRow))
      .flatMap(optionalAsset -> optionalAsset.map(Stream::of).orElseGet(Stream::empty))
      .collect(Collectors.toList());
    
    if (potentialAssetList.isEmpty()) {
      logger.error("Non match");
      return Optional.empty();
    }
    if (potentialAssetList.size() > 1) {
      logger.error("More that one match");
      return Optional.empty();
    }
    return Optional.of(potentialAssetList.get(0));
  }
}
```





