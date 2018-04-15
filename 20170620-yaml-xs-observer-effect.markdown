Title: YAML::XS and observer effect
Tags: Perl

> In science, the term observer effect means that the act of observing will influence the phenomenon being observed.

YAML::XS is modifying Perl structures when you look into it (dump). Let me show you how.

[cut]

Imaging you have a structure `my $foo = {bar => 1}`. And now we dump it with `YAML::XS`:

```
perl -MYAML::XS -E 'my $foo = {bar => 1}; say YAML::XS::Dump($foo)'

---
bar: 1
```

Nothing's wrong you think. But let's dump it with JSON:

```
perl -MYAML::XS -MJSON -E 'my $foo = {bar => 1}; say YAML::XS::Dump($foo); say JSON::encode_json($foo)'
---
bar: 1

{"bar":"1"}
```

As you can see `1` became `"1"`. If we you use `Devel::Peek` we can see that:

```
perl -MDevel::Peek -MYAML::XS -e 'my $foo = 1; print Devel::Peek::Dump($foo); YAML::XS::Dump($foo); print Devel::Peek::Dump($foo)'

SV = IV(0x55fc5098e810) at 0x55fc5098e820
  REFCNT = 1
  FLAGS = (IOK,pIOK)
  IV = 1
SV = PVIV(0x55fc5098a960) at 0x55fc5098e820
  REFCNT = 1
  FLAGS = (IOK,POK,pIOK,pPOK)
  IV = 1
  PV = 0x55fc50a68fa0 "1"\0
  CUR = 1
  LEN = 10
```

Our variable now has new flags. It turns out that `SvPV_nomg` in `YAML::XS` code adds the string representation for the number and creates side effects.

The fix has been proposed by me in this pull request <https://github.com/ingydotnet/yaml-libyaml-pm/pull/55>, but unfortunately it hasn't been merged so far...
