# JMESPath Support

[JMESPath](https://jmespath.org/tutorial.html) is a query language for JSON.

Glaze currently has limited support for partial reading with JMESPath. Broader support will be added in the future.

- Compile time evaluated JMESPath expressions offer excellent performance.

Example:

```c++
struct Person
{
   std::string first_name{};
   std::string last_name{};
   uint16_t age{};
};

struct Family
{
   Person father{};
   Person mother{};
   std::vector<Person> children{};
};

struct Home
{
   Family family{};
   std::string address{};
};

suite jmespath_read_at_tests = [] {
   "compile-time read_jmespath"_test = [] {
      Home home{.family = {.father = {"Gilbert", "Fox", 28},
                           .mother = {"Anne", "Fox", 30},
                           .children = {{"Lilly"}, {"Vincent"}}}};

      std::string buffer{};
      expect(not glz::write_json(home, buffer));

      std::string first_name{};
      auto ec = glz::read_jmespath<"family.father.first_name">(first_name, buffer);
      expect(not ec) << glz::format_error(ec, buffer);
      expect(first_name == "Gilbert");

      Person child{};
      expect(not glz::read_jmespath<"family.children[0]">(child, buffer));
      expect(child.first_name == "Lilly");
      expect(not glz::read_jmespath<"family.children[1]">(child, buffer));
      expect(child.first_name == "Vincent");
   };

   "run-time read_jmespath"_test = [] {
      Home home{.family = {.father = {"Gilbert", "Fox", 28},
                           .mother = {"Anne", "Fox", 30},
                           .children = {{"Lilly"}, {"Vincent"}}}};

      std::string buffer{};
      expect(not glz::write_json(home, buffer));

      std::string first_name{};
      auto ec = glz::read_jmespath("family.father.first_name", first_name, buffer);
      expect(not ec) << glz::format_error(ec, buffer);
      expect(first_name == "Gilbert");

      Person child{};
      expect(not glz::read_jmespath("family.children[0]", child, buffer));
      expect(child.first_name == "Lilly");
      expect(not glz::read_jmespath("family.children[1]", child, buffer));
      expect(child.first_name == "Vincent");
   };
};
```
